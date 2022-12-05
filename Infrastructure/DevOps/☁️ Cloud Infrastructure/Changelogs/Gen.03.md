# Mission Statement

This revision is easily the largest change since our initial migration to the cloud, having nearly two years of experience with Azure by this point we understood where our pain points were and worked to mitigate them and significantly enhance platform stability and security. These enhancements included the following:

-   Migration from Azure West Europe (Amsterdam) to Azure UK South (London)
-   Azure Firewall in a Hub/Spoke model
-   Using Azure CNI instead of Weave Net
-   Breaking every environment out into its own cluster
-   First implementation of GitOps via Flux v1
-   Switch to Containerd
-   Dedicated Networks for Kubernetes with a three tier design; [“worker”, “controller”, “etcd”].
-   Postgres via Azure Database for PostgreSQL Single Server (Single Server product was actually a HA pair).
-   Highly available RabbitMQ Cluster moved back outside of Kubernetes

# Tooling

-   Terraform
-   Chef

# Software

-   Ubuntu 16.04
-   Bifrost v2
-   Containerd
-   Flux v1
-   Azure CNI

# Azure Components Used

-   Azure Virtual Machine
-   Azure Firewall
-   Azure Load Balancer (Premium)
-   Azure Database for PostgreSQL Single Server