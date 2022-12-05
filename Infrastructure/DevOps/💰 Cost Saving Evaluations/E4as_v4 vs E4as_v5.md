The v4 Series has a `AMD EPYC 7452 32-Core Processor` while the v5 runs with a `AMD EPYC 7763 64-Core Processor`. However, each VM is limited to 4 Cores. I've also tested with an ARM64 CPU, Azure is cagey about telling us what exactly it is however.

## Pricing
Pricing, the v4 comes in at $216.08/month, the v5 at $194.18/month, and the ARM64 at $173.01.

## Performance
### Fabrication Process
The v4 CPU is from 2019 and uses a 7nm process, while the v5 is from 2021 it also uses the same process size. This suggests that any improvements are likely coming from time optimising the design of the EPYC architecture rather than physical hardware gains. Once again, I'm unable to figure out the difference in fabrication process for the ARM64 chip.

### Sysbench
#### CPU
Using Sysbench we can see that the v4 achieves `3461.92` events per second using all four threads, while the v5 gets `8290.75`. Making it 2.39 times faster in raw integer operations than the v4 counterpart. ARM64 achieves an impressive `13481.43`, making it a huge 3.89 times faster.

#### Memory
Using Sysbench again we'll now compare memory performance, the v4 achieves `14632287.32` memory operations per second, while the v5 only manages `12419347.42` - this is a .1 difference in performance, meaning both chips likely use the same memory controller and ECC DRAM. **ARM64 achieves `61615196`,** thats 4.2 times faster than the AMD counterparts.

#### Python RSG and Disk Access
For this I've written an intentionally inefficient script that opens and closes a file a bunch of times while writing a random set of characters to said file.

```python
import random
import string
from tqdm import tqdm

for _ in tqdm(range(500)):
    characters = string.printable
    random_string = ''.join(random.choice(characters) for i in range(4096000))

    with open("/tmp/testfile", "w") as f:
        f.write(random_string)
```

The v4 completed this test in 13:40 (1.64s/it) while the v5 managed to do it in 10:57 (1.32s/it). This is a 24.81% real world difference in Python performance. Unfortunately, this was the first loss for the ARM64 server, managing the test in 15.58 seconds (1.87s/it). It would seem IO is significantly worse on ARM64.  

#### Python API
Next, I wrote a very basic API that only tries to reply as fast as possible with as little logic as possible. Code here:

```python
from wsgiref.simple_server import make_server
import falcon

app = falcon.App()


class Healthz:
    def on_get(self, req, resp):
        resp.status = falcon.HTTP_204healthz = Healthz()


app.add_route("/healthz", healthz)

if __name__ == "__main__":
    with make_server("", 6502, app) as httpd:
        print("Serving on port 6502")
        httpd.serve_forever()
```

I used `wrk` to benchmark this with: `wrk -t4 -c400 -d30s [http://localhost:6502/healthz](http://localhost:6502/healthz "http://localhost:6502/healthz")`  - v4 managed 76,368 requests in 30s, while v5 managed 97,098 requests. Thats a 27% difference in favour of the v5. Unfortunately, the ARM64 server lost out here also only managing 47,775 requests.

## Summary

So, the E4as_v5 is about 25% faster for Python workloads than the E4as_v4. However, while the ARM64 Server absolutely trashes the AMD servers in compiled workloads, our Python workloads are as much as 50% slower than the v5 instance, so hopefully we see some further optimisations in Python for ARM chips over time.

Cost Breakdown, we have 41 Servers in total in our AKS Clusters,
| Name      | VM Cost | Disk Cost | Count | Total     | With Reservations |
| --------- | ------- | --------- | ----- | --------- | ----------------- |
| E4as_v4   | $216.08 | $0.00     | 41    | $8,859.28 | $3,366.53         |
| E4as_v5   | $194.18 | $25.17    | 41    | $8,993.35 | $3,025.32         |

So, we may still save a small amount of money (once you factor in disks) by moving to V4, but it's only about $1000 at this point, that _might_ be worth it, but for a 25% performance loss on all clusters, I'm not convinced. Nathan and Mark, you decide. But I think we're better putting pressure on Microsoft to add Ephemeral Disk support to v5 nodes and biding our time on cheaper compute, in the mean time we can reduce our AKS node disks from 128GB to 32GB, bringing the $1000 a month bill down to $250 (£203) for zero loss of service, performance, or functionality.