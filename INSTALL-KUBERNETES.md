# Installing the CLI Tools (kubectl, eksctl, AWS CLI)

You need two command-line tools on your own machine:

- `kubectl`: talks to a Kubernetes cluster.
- `eksctl`: creates and deletes Amazon EKS clusters with one command.

You also need the AWS CLI configured with credentials that can create an EKS cluster (your admin credentials, separate from the limited Jenkins key). Install steps for the AWS CLI are in the section below; then run `aws configure`.

You also need Docker installed and running. If you do not have it yet, install Docker Desktop on Windows or macOS (https://www.docker.com/products/docker-desktop), or Docker Engine on Linux (https://docs.docker.com/engine/install). On the EC2 server, the Docker install steps are in `RUN-ON-AWS.md`.

On an ARM machine, replace `amd64` with `arm64` in the Linux URLs.

## Windows

With `winget`:

```powershell
winget install -e --id Kubernetes.kubectl
winget install -e --id Weaveworks.eksctl
```

Or with Chocolatey:

```powershell
choco install kubernetes-cli -y
choco install eksctl -y
```

## Linux

### kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client
```

### eksctl

```bash
curl -sSL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz" \
  | sudo tar -xz -C /usr/local/bin
eksctl version
```

## macOS (for reference)

```bash
brew install kubectl eksctl
```

## Install the AWS CLI

You run `eksctl` and `kubectl` against AWS from your own machine, so you need the AWS CLI here too.

Windows (winget), or download the MSI from https://awscli.amazonaws.com/AWSCLIV2.msi :

```powershell
winget install -e --id Amazon.AWSCLI
```

Linux:

```bash
curl -fsSL "https://awscli.amazonaws.com/awscli-exe-linux-$(uname -m).zip" -o awscliv2.zip
unzip awscliv2.zip
sudo ./aws/install
```

macOS:

```bash
brew install awscli
```

Then configure it with credentials that can create an EKS cluster. These should be your own AWS account credentials (admin or equivalent), not the limited `jenkins-ecr` key, which can only touch ECR.

```bash
aws configure
```

Enter your Access key ID, Secret access key, default region (for example `us-east-1`), and output format (`json`).

## Verify

```bash
kubectl version --client
eksctl version
aws sts get-caller-identity
```

The last command confirms the AWS CLI knows who you are. If it errors, run `aws configure`.
