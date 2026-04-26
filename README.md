## 📌 Description

TerraCloud is an infrastructure-as-code project that leverages Terraform, Packer, and Ansible to automate
the provisioning and configuration of a scalable web application on AWS environment.

The complete IaC stack! Create custom AMIs with Packer, provision infrastructure with Terraform, and configure servers with Ansible. 
From basic deployments to enterprise architectures with high availability, load balancers, and bastion hosts.

## ⚙️ Setup Installation of the Required Tools

**AWS CLI**

## macOS
brew install awscli

## Ubuntu/Debian
sudo apt update && sudo apt install awscli

## pip
pip install awscli

###

**Terraform**

## tfenv (Terraform version manager) – RECOMMENDED
## macOS
brew install tfenv

## Ubuntu/Debian
git clone https://github.com/tfutils/tfenv.git ~/.tfenv
echo 'export PATH="$HOME/.tfenv/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

## Install and use the latest version of Terraform
tfenv install latest
tfenv use latest

## Alternate: Direct Install
## macOS
brew install terraform

## Ubuntu/Debian
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform

## tfenv quick use:
## List available Terraform versions
tfenv list-remote

## Install a specific Terraform version
tfenv install 1.12.0

## Switch to a specific version
tfenv use 1.12.0

## Install version defined in .terraform-version file
tfenv install

###

**Ansible**

## macOS
brew install ansible

## Ubuntu/Debian
sudo apt update && sudo apt install ansible

## pip
pip install ansible

###

**Packer** 

## macOS
brew install packer

## Ubuntu/Debian
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt update && sudo apt install packer

## Or download from https://www.packer.io/downloads

## 📝 How to use the provision

AWS Configuration

Configure your AWS credentials with profiles:

## Configure AWS CLI with a named profile
aws configure --profile yourprofile

## Enter when prompted:
## AWS Access Key ID: your-access-key
## AWS Secret Access Key: your-secret-key
## Default region name: us-east-1
## Default output format: json

## Use the profile in commands
aws s3 ls --profile yourprofile
aws sts get-caller-identity --profile yourprofile

## Or set it as default for the session
export AWS_PROFILE=yourprofile

## Alternative: set environment variables directly
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
export AWS_DEFAULT_REGION="us-east-1"

SSH Key Configuration (Required only for Packer)
Note: SSH keys are only required for (creating an AMI with Packer). Terraform automatically creates its own keys. 
Why Packer needs an SSH key:

Packer uses SSH to connect to the temporary EC2 instance during AMI building

Ansible (via Packer) needs SSH access to configure the instance

After the AMI is created, the key is no longer needed for running instances

Generate SSH key pair for Packer:

## Generate ED25519 key (recommended)
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519

## Or RSA key
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa

## The key will be used in Lab 3 when running:
## cd packer && ./build.sh

Verification
Check if all tools are installed correctly:

## Check versions
aws --version
terraform --version
ansible --version
packer --version

## Test AWS connectivity with a profile
aws sts get-caller-identity --profile yourprofile

## Or if AWS_PROFILE is set
export AWS_PROFILE=yourprofile
aws sts get-caller-identity

## 🚀 Provisioning

**Terraform** 

Step 1: Configure S3 Backend (First Run)

Before provisioning the infrastructure, you need to create an S3 bucket to store the Terraform state remotely and securely.

cd terraform

## Option 1: Use environment variables (recommended)
BUCKET=terra-cloud-lab REGION=us-east-1 PROFILE=yourprofile ./bootstrap.sh

## Option 2: Interactive mode (you will be prompted for values)
./bootstrap.sh

Step 2: Provision Infrastructure

## Still inside terraform

## Option 1: Use the same environment variables (recommended)
PROFILE=yourprofile BUCKET=terra-cloud-lab REGION=us-east-1 ./provision.sh

## Option 2: If you already exported the variables, just run:
./provision.sh

## Follow the prompts to confirm infrastructure creation

Verification

## Check infrastructure outputs

terraform output

## Test SSH connectivity (confirms the instance is accessible)

eval "$(terraform output -raw ssh_connection_command)"

## Or simply copy and run the displayed command manually:
## terraform output ssh_connection_command

## SSH to bastion host
eval "$(terraform output -raw ssh_bastion_command)"

## SSH to web servers via bastion (check available commands)
terraform output ssh_web_servers_commands

## Connect to web server 1 (copy and run the first command from the list above)
## Connect to web server 2 (copy and run the second command from the list above)

## Test HTTP port (there is no web server yet, so it will fail)
curl "http://$(terraform output -raw instance_public_ip)"

## Expected: timeout or "Failed to connect" — this is normal, since no web server is installed yet

Clean

./clean.sh - will result in aws config being cleared off

Destroy

./destroy.sh - will result in all tf state and aws provision destroyed

## Server Configuration with Ansible

Objective: Configure the EC2 instance with Docker and deploy a containerized website.

What You Will Configure:

- Install Docker Engine

- Configure Docker Compose v2

- Deploy an Nginx container

- Custom website with lab information

- Health monitoring

- Deployed Services

- Docker daemon

- Containerized Nginx web server

- Custom HTML website

- Container health checks

Instructions:

## Make sure the infrastructure is running
cd terraform
terraform output instance_public_ip

## Configure the server
cd ../ansible

## Edit inventory.ini with your instance IP
./configure.sh

Verification:

## Access the website
curl http://<INSTANCE_IP>

## Expected: Custom HTML page with web information

## Creating a Custom AMI with Packer

cd packer

./build.sh

## Enter the SSH key path when prompted (default: ~/.ssh/id_ed25519)
## Note the AMI ID from the output

## Instructions for the terraform with ami (web server ready): 

cd terraform

## If this is your first time, run:
## BUCKET=terra-cloud-lab REGION=us-east-1 PROFILE=yourprofile ./bootstrap.sh

## Provision using the custom AMI by name (not ID)
PROFILE=yourprofile BUCKET=terra-cloud-lab INSTANCE_AMI=Myimage-* AMI_OWNER=self ./provision.sh
## The wildcard (*) matches the timestamp, finding your most recent AMI

## Or specify the exact name if you have multiple AMIs
PROFILE=yourprofile BUCKET=terra-cloud-lab INSTANCE_AMI=Myimage-202613456 AMI_OWNER=self ./provision.sh

Verification:

## Check all outputs
terraform output

## Access website via load balancer
curl "$(terraform output -raw load_balancer_url)"

## SSH to bastion host
eval "$(terraform output -raw ssh_bastion_command)"

## SSH to web servers via bastion (check available commands)
terraform output ssh_web_servers_commands

# Connect to web server 1 (copy and run the first command from the list above)
# Connect to web server 2 (copy and run the second command from the list above)

###

Reference: Manual SSH Access via Bastion (ProxyCommand)

If you prefer to build the command manually or understand how it works:

## Get required IPs
BASTION_IP=$(terraform output -raw bastion_public_ip)
WEB_IPS=($(terraform output -json web_private_ips | jq -r '.[]'))
KEY_PATH=$(terraform output -raw private_key_path)

## SSH to web server 1 using bastion as proxy
ssh -o IdentitiesOnly=yes -i "$KEY_PATH" \
  -o ProxyCommand="ssh -o IdentitiesOnly=yes -i $KEY_PATH -W %h:%p ec2-user@$BASTION_IP" \
  ec2-user@${WEB_IPS[0]}

## SSH to web server 2 using bastion as proxy
ssh -o IdentitiesOnly=yes -i "$KEY_PATH" \
  -o ProxyCommand="ssh -o IdentitiesOnly=yes -i $KEY_PATH -W %h:%p ec2-user@$BASTION_IP" \
  ec2-user@${WEB_IPS[1]}

Example: Customizing Modules

Change number of web servers:

PROFILE=yourprofile BUCKET=terra-cloud-lab INSTANCE_AMI=Myimage-* AMI_OWNER=self WEB_SERVER_COUNT=3 ./provision.sh

Disable NAT Gateway (cost saving):

PROFILE=yourprofile BUCKET=terra-cloud-lab INSTANCE_AMI=Myimage-* AMI_OWNER=self ENABLE_NAT_GATEWAY=false ./provision.sh

## Note: Without a NAT Gateway, web servers will not have internet access

Create without bastion host:

PROFILE=yourprofile BUCKET=terra-cloud-lab INSTANCE_AMI=Myimage-* AMI_OWNER=self CREATE_BASTION=false ./provision.sh

## Note: Without a bastion host, you will not be able to SSH into private web servers
