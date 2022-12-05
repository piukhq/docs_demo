All of the below commands should be executed as root via `sudo -s`

Install `docker`, `docker-compose`, and `python3-pip`:
```shell
apt -y install docker docker-compose python3-pip
```

Install `pipenv`:
```shell
$ pip3 install pipenv
```

Create a directory for Prefect in `/opt`
```shell
$ mkdir /opt/prefect
$ cd /opt/prefect`
```

Install Prefect using `pipenv`:
```shell
$ pipenv install "prefect<3" adlfs
```

Setup Systemd:
```shell
$ cat > /etc/systemd/system/prefect.service <<EOF
[Unit]
Description=Prefect

[Service]
ExecStart=pipenv run prefect orion start
WorkingDirectory=/opt/prefect

[Install]
WantedBy=multi-user.target
EOF

$ systemctl daemon-reload
$ systemctl start prefect.service
```

---
# Original Documentation from Lumilinks
The following is a guide to set up all the infrastructure required to run Prefect on a VM.

### VM
- Create a VM with an ubuntu image. Keep SSH ports open for now.
	- Our setup utilises Azure’s Standard B2ms (2 vcpus, 8 GiB memory). Only a small amount of compute power is required, and only for a small amount of time. Therefore a small burstable VM is well suited.
- SSH onto the VM and run the following commands:
    - `sudo apt-get update; sudo apt-get upgrade`
    - `sudo apt install docker-compose; sudo groupadd docker; sudo usermod -aG docker ${USER}`
    - Log out of the VM, then log in again and check that running `docker images` doesn’t raise any errors.
    - `sudo apt install python3-venv`
    - `python3 -m venv venv`
    - `source venv/bin/activate` (Whenever logging into the VM, run the `source venv/bin/activate` command to reactivate the virtual environment.)
    - `pip install prefect`
    - `prefect backend server`
    - Create a file called `~/.prefect/config.toml` and copy in the following:
```toml
[server]
[server.ui]
apollo_url = "http://<ip of vm>:4200/graphql"
[cloud]
api = "http://<ip of vm>:4200"
```
- `prefect server start --expose`
- Whitelist your IP to access port 8080 and 4200, and you will be able to access the UI at the IP of the VM on port 8080 

### Postgres

Prefect Server requires a postgres DB to run. If none is provided, it will spin up a dockerised postgresql. As this is impermanent, we need to create a permanent azure postgres server.

Create a flexible azure postgres server, and make it publicly available. Ensure the admin password contains a capital letter, lowercase letter, a number and an underscore. Don’t use any other special character as this will break the connection string.

Whitelist the IP of the prefect server, and change the server parameters: under azure.extensions tick PGCRYPTO and PG_TRGM.

### Starting the server

Use the following command to start the server:

`prefect server start --postgres-url "postgresql://<pg admin name>:<pg admin password>@<pg server address>/postgres?sslmode=require" -ep --expose`

This command will start the server using our postgres server, and exposes the endpoint so we can connect to it via port 8080.

### Start an agent

Once the server is up and running, an agent is required to run the flows. To create a docker agent, run the following command from in the venv:

`nohup prefect agent docker start -n docker-agent-prod -l prod --env dbt_environment=prod > ~/.cache/prefect-agent-prod-logs &`

The `-l` flag provides a label, which determines what flows are run by the agent. Any logs are sent to the `.prefect-agent-prod-logs` file. You can check that an agent is running from the agent tab in the Prefect server UI.

The `--env` flag provides the dbt_environment variable which can be picked up by the flow as an environment variable and used to set the dbt parameters

You can check if there’s an agent running using `pgrep –a 'prefect'`

You can kill the agent using `kill -9 <pid>` with the PID value being the first number returned from the pgrep command.

### Registering a flow

To register a flow, pull down the Bink repo onto the VM. This might require setting up an ssh key pair

In the Prefect directory within the repo, run `dbt_environment=prod prefect register --project Bink -l prod -p run_elt.py`

This will build a docker image containing the dbt code, which can be run on the docker agent (with the tag of 'prod') whenever triggered by Prefect server.

If there is a code change to DBT, or the Prefect run_elt file is altered, the flow will need reregistering.

Note: as the flow requires the use of secrets, these should be added to the config.toml file under the heading `context.secrets`:
```toml
[context.secrets]
bink_airbyte_ip="<airbtye vm ip>"
bink_snowflake_account="<snowflake id.eu-west-2.aws>"
bink_snowflake_password="<transform user password>"
```