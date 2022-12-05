# Mission Statement

This revision of our infrastructure was the migration away from Bifrost, our in-house distribution of Kubernetes, to Azure Kubernetes Service. Continuing to support the rapidly changing Azure ecosystem was becoming increasingly complex and took up a large amount of DevOps resources. AKS had finally matured to a product that was close enough to Bifrost that we could migrate with minimal friction.

Given Bifrost, which was deployed via Chef, was no longer in the picture we also took the decision to migrate from Chef to Ansible. Recruiting for the DevOps role often required some experience with Ruby, or some willingness to learn it - migrating to Ansible removed this limitation.

# Tooling

-   Terraform
-   Ansible

# Software

-   Ubuntu 18.04 (AKS uses older components than Bifrost, however, AKS 1.25 will migrate to Ubuntu 22.04)
-   Flux v2