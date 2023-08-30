Order:
- Who we are including scale
- Why we use automation
- Tools and why
- Details

Who we are - 10m
---------------------

Presentor introductions
UFIT / ICT
Name team members
What do we do:
- Manage compute infrastructure from the hardware to the hypervisor and beyond
- Compute usage is split between Enterprise and Hosting
- Enterprise
	- Help architect solutions for other teams which sometimes leads to standard operating procedures for other teams
- Hosting
	- Provided a self-service platform for non-UFIT customers to get various hosting services
	- list services

Scale
- Private Cloud
	- Availability zone structure
	- IaaS
		- VMware
		- number of ESXi hosts and clusters
		- number of VMs ( Enterprise and Hosting )
	- PaaS
		- Apache/IIS
		- Databases
		- File shares
- Public Cloud
	- AWS
	- Azure
	- GCP

Why we use automation - 5m
-------------------------------------------

Theme: Scale and consistency

At the pace of incoming projects and requests and at our scale, there really is no place for ClickOps.
To have a consistent environment you need consistent processes.

Automate all of this using Infrastrcuture as Code tools and Continuous Deployment Tools
Tools and Why - 10m
-------------------------------------

Introducing the Terrable stack + GitLab + Vault

Terraform + Ansible = Terrable but Awesome!

Terraform:
- Hashicorp Terraform is an Infrastructure as Code ( IaC ) tool that has lots providers that can manage most any infrastructure.
- Uses a desired state approach to deploy infrastructure. So not a step by step plan but the actual end goal. Will then create a dependency graph to generate a plan to get to that desired state. Will check current state vs last known state and use the plan to figure out what steps need to be taken to reach desired state.
- Really good for creating the fundamental infrastructure components:
	- Deploy a VM
	- Manage VM configurations like adding more disk or changing network interface attributes
	- Manage DNS entries
	- Manage DHCP assignments
- Not very good for managing the operating system or the applications within the OS
- Capability to create modules as reusable components with a clean interface - more on this later

Ansible:
- Ansible is also an IaC tool but instead of using desired state, it uses a procedural process ( sequential step by step ) process to achieve its goal. You tell it what to do and in what order and it will attempt to get it done. If a step fails then it will potentially stop.
- Has lots of modules ( called collections ) to manage OS components:
	- Manage Users
	- Deploy OS packages
	- Create filesystems and mount points
	- Manage docker containers
	- Manage configuration of various software packages
- Cross OS support
	- Pretty much all Linux distributions
	- Windows
- Also has the capability to create reusable components called roles

Vault:
- What are secrets:
	- Usernames and Passwords
	- TLS Certificates and Private Keys
	- API Tokens
- We all have secrets but we need to share them in a secure way
- We also need a way to rotate those secrets in a secure way
- Hashicorp Vault is a secrets management tool
	- Can store static secrets
		- username and passwords
	- Can interact with authentication systems and generate dynamic credentials
		- Active Directory
		- AWS IAM
		- Azure AD
- Vault has a built-in policy engine that is used to declare what can access which secrets
- Vault has a concept of an AppRole which can be assigned to a service or machine so that it can access secrets as well
	- GitLab has an AppRole that gives it access to secrets stored in Vault
- Both Terraform and Vault have native support to access Vault
- We use Terraform to manage our Vault deployment

Use Terraform to deploy the base infrastructure components with some exceptions:
- VMs in VMware
- S3 buckets in AWS
- App Services in Azure

Use Ansible to manage and deploy applications on top of the those components ( when needed )

Use Vault to store and manage secrets

GitLab is the glue to all this code.
GitLab is a lot more than just a git repo with a pretty frontend:
- Issue tracking
- Merge Request handling with approvals
- Deployment Pipelines triggered based on various events

Deployment pipelines consist of a all the steps required to complete the task at hand
Pipelines are modular where you can specify that certain stages can depend on other stages so that each part runs on its own in serial or some stages can be run in parallel

GitLab component reuse:
- GitLab has the capability to create project templates that can be reused by multiple people
- Pipeline stages can be imported from a central repository

We use GitLab for Continuous Deployment. When a commit happens, depending on the branch or which directories within the repo are modified, a pipeline is kicked off which uses Terraform, Ansible, or both. During that pipeline run, Vault is queried for any required secrets that are needed for that run.

Details - 15m
-----------------------