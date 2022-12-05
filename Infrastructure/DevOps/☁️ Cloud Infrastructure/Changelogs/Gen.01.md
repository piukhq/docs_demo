# Mission Statement

The first revision of our Cloud Infrastructure was the migration away from our original Data Centre (UK Fast) to Microsoft Azure. The design fundamentals remain the same as UKFast, just deployed to the Cloud instead - all environments ran from the same server.

# Tooling

-   rake (Ruby Make)
-   Chef

# Software

-   Ubuntu 16.04
-   kubeadm
-   Docker 18.03
-   Weave Net

# Azure Components Used

-   Azure Virtual Machine
-   Azure Load Balancer (Basic)
-   Azure Application Gateway

# Network Design

The entire network consisted of a single Virtual Network with multiple subnets. Azure Basic Load Balancers were used.