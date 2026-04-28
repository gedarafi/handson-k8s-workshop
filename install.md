# Preparation Session – "Hands-on Kubernetes as a User" Workshop

Welcome to the "Hands-on Kubernetes as a User" workshop! We'll be using [k3d](https://k3d.io/stable/), one of the many single-node Kubernetes flavours available. There are alternatives such as [minikube](https://minikube.sigs.k8s.io/) and [kind](https://kind.sigs.k8s.io/), but they won't be covered here. This short guide walks you through installing k3d and its prerequisites; most of the content is also available in the [official k3d documentation](https://k3d.io/stable/#installation).

If you run into issues, join the online help session **Tuesday, April 28, 2026, 13:00–15:30** (before the actual workshop). If you're properly registered, the link should be in the e-mail you got this document with.

# Requirements and Instructions

The main rule: **don't use plain Windows**. Windows Subsystem for Linux (WSL), macOS, or any Linux distribution (Debian, Ubuntu, Arch) will all work fine.

k3d has two main software requirements: **Docker** and **kubectl**. You'll also need administrative access on your machine. If you're reinstalling Docker, make sure any existing Docker installation is fully removed first.

## Preparation: Using Linux (and WSL)

### Installing Docker

Installing Docker on Linux depends on your distribution, since package managers differ. This guide focuses on Ubuntu, which uses the Aptitude (`apt`) package manager and is one of the most popular distributions. For other distributions, refer directly to the [official documentation](https://docs.docker.com/engine/install/).

Copy and paste the following into your terminal:

```bash
# Add Docker's official GPG key
sudo apt update
sudo apt install ca-certificates curl -y
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to apt sources
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

# Install Docker
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

### Installing kubectl

`kubectl` isn't in Ubuntu's default repositories, so you'll need to add the Kubernetes apt repository first:

```bash
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl gnupg

# Add the Kubernetes GPG key
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.35/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Add the Kubernetes repository
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' \
  | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Install kubectl
sudo apt update
sudo apt install -y kubectl
```

Great! Now go to the k3d installation!

## Preparation: Using macOS

### Easiest option: Homebrew

If you have [Homebrew](https://brew.sh) installed, a single command is enough:

```bash
brew install docker kubectl
```

Done, if you were successful then you don't need to do any of the steps below!

### Installating Docker (without Homebrew)

Download Docker Desktop [here](https://desktop.docker.com/mac/main/arm64/Docker.dmg?utm_source=docker&utm_medium=webreferral&utm_campaign=docs-driven-download-mac-arm64) (for Macs with Apple Silicon). Installation is straightforward — just go with the **Recommended Settings**.

### Installating kubectl (without Homebrew)

Download the binary:

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/arm64/kubectl"
```

Then set permissions and move it somewhere on your `PATH`:

```bash
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
sudo chown root: /usr/local/bin/kubectl
```

## Testing the prerequisites (all operating systems)

Test kubectl with:

```bash
kubectl cluster-info
```

You'll likely see something like:

```
The connection to the server <server-name:port> was refused - did you specify the right host or port?
```

That's expected as we haven't installed k3d yet, but it confirms kubectl is working.

For Docker, run:

```bash
sudo docker run hello-world
```

This downloads the container (if it isn't already present), prints a "Hello World" message, and exits.

## Installing k3d (all operating systems)

Installation is a one-liner:

```bash
curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash
```

The script requires `sudo`, so you'll be asked for your password.

Once installed, create a cluster:

```bash
sudo k3d cluster create mycluster
```

Then check that it's running:

```bash
sudo kubectl get pods -A
```

If you see a list of pods, everything is working — and we look forward to seeing you at the workshop!

## Running kubectl without sudo

You probably don't want to prefix every `kubectl` command with `sudo`. To fix that:

1. Copy the `.kube` folder from `/root/` to your home directory:
   ```bash
   sudo cp -r /root/.kube ~/.kube
   ```
2. Change ownership from root to your current user:
   ```bash
   sudo chown -R $USER:$USER ~/.kube
   ```

Now test kubectl as your normal user:

```bash
kubectl get nodes
```