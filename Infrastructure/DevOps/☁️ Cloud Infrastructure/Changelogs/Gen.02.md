# Mission Statement

This revision of our Cloud Infrastructure was a fast follow to 1st generation which took approximately six months to develop. Fundamental changes were made including the writing of the first version of Bifrost (In House Kubernetes Distribution), and a “Run everything in Kubernetes” mentality, unfortunately not ‘everything’ could be moved as Bifrost hadn’t fully solved Persistent Storage in Azure yet. Much like First Generation infrastructure, all environments existed on a single cluster - however, this cluster was considerably more heavily loaded as Redis, RabbitMQ, etc all ran in said cluster instead of on dedicated VMs.

# Tooling

-   Terraform
-   Chef

# Software

-   Ubuntu 16.04
-   Bifrost v1
-   Docker 17.03
-   Weave Net

# Azure Components Used

-   Azure Virtual Machine
-   Azure Load Balancer (Premium)

# Network Design

Mostly the same as Gen.1.
