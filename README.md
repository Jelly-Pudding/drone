# Drone Overview

Drone is an open-source continuous integration (CI) and delivery (CD) platform. It is designed to be lightweight and easy to use, with a simple configuration file and a powerful API.

With Drone, you can automate the build, test, and deployment of your software projects. It integrates with a wide range of tools and services, including version control systems like Git, container registry platforms like Docker Hub, and deployment tools like Kubernetes.

Using Drone, you can define build pipelines in a configuration file, and Drone will automatically run the defined steps in the pipeline whenever code is pushed to the repository. This allows you to automate your build, test, and deployment process, freeing up time and resources to focus on other tasks.

Drone is highly flexible and can be used for a variety of software projects, including web applications, mobile apps, and infrastructure as code. It is popular among developers and DevOps teams because of its simplicity, ease of use, and extensibility.

Drone specifically consists of a `Server` and one or many `Runners`. `Runners` are standalone daemons that poll the server for pending pipelines to execute.

This repository outlines the process for setting up both the Drone `Server` and a Drone `Runner` for Docker and Kubernetes. Click the links below to navigate to the desired section: 

* [Drone Server Installation](#drone-server-installation)
* [Install a Runner](#install-a-docker-runner-on-linux)
* [Managing Drone Secrets](#secrets-per-repository)

# Drone Server Installation

## Ngrok Installation
Your Drone server will need to be publicly accessible. In this example, `ngrok` will be used. Download instructions for Linux can be found [here](https://ngrok.com/download). You can also follow the below steps:

> 1. Run `wget https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-v3-stable-linux-amd64.tgz`.
> 2. Extract ngrok with `sudo tar xvzf your/present/directory/file -C /usr/local/bin`.
> 3. [Sign up](https://ngrok.com/) with ngrok in order to get an authtoken.
> 4. Add the authtoken with ngrok config add-authtoken token-goes-here
> 5. Start ngrok with this command: `ngrok http 80`.
> 6. You will be publicly available at the URL underlined below: 
![image](./pictures/ngrok.png)


## Drone Installation for GitHub
You can follow the instructions on [Drone's official documentation page](https://docs.drone.io/server/provider/github/) for installing a Drone server for GitHub. Alternatively, you can follow the steps below:

> 1. [Create a GitHub OAuth application](https://docs.github.com/en/developers/apps/building-oauth-apps/creating-an-oauth-app). You can name the application `Drone`, and for the `Homepage URL` and the `Authorization callback URL`, put in the publicly accessible ngrok URL followed by `/login` at the end.
> 2. Make a note of the `Client ID` and especially the `Client Secret` which you won't be able to see again. 
> 3. In addition to these secrets, create a shared secret by typing this command into the terminal: `openssl rand -hex 16`. Also make a note of this secret.
> 4. Install Docker. Remember you might need to run an update and an upgrade first.
> 5. Pull the Drone docker image with this command: `docker pull drone/drone:2`. 
> 5. Execute this docker run command:
```bash
docker run \
  --volume=/var/lib/drone:/data \
  --env=DRONE_GITHUB_CLIENT_ID=your-id \
  --env=DRONE_GITHUB_CLIENT_SECRET=super-duper-secret \
  --env=DRONE_RPC_SECRET=super-duper-secret \
  --env=DRONE_SERVER_HOST=https://211f-2-121-41-213.eu.ngrok.io \
  --env=DRONE_SERVER_PROTO=https \
  --publish=80:80 \
  --publish=443:443 \
  --restart=always \
  --detach=true \
  --name=drone \
  drone/drone:2
```
The `DRONE_RPC_SECRET` is the secret created by running the command `openssl rand -hex 16`. The server host is the publicly accessible ngrok URL.
> 6. You will be able to access the Drone server by typing the ngrok URL into your browser. 

# Install a Docker runner on Linux

Official documentation for installing the Docker runner on Linux can be found [here](https://docs.drone.io/runner/docker/installation/linux/).

> 1. Pull the public runner image: `docker pull drone/drone-runner-docker:1`.
> 2. Run the below command, replacing the relevant configuration details:
```bash
docker run --detach \
  --volume=/var/run/docker.sock:/var/run/docker.sock \
  --env=DRONE_RPC_PROTO=https \
  --env=DRONE_RPC_HOST=211f-2-121-41-213.eu.ngrok.io \
  --env=DRONE_RPC_SECRET=44711333dd6ed9a9bd7c3aa1c8d72739 \
  --env=DRONE_RUNNER_CAPACITY=2 \
  --env=DRONE_RUNNER_NAME=my-first-runner \
  --publish=3000:3000 \
  --restart=always \
  --name=runner \
  drone/drone-runner-docker:1
```
The `DRONE_RPC_HOST` must point to your ngrok URL and it must not have the `https://` at the start. The `DRONE_RPC_SECRET` is the one created from the `openssl rand -hex 16` command. This must match the secret defined in your Drone server configuration.
> 3. Use the `docker logs container-id-here` command to view the logs. Confirm whether the runner established a connection with your Drone server. The below image acts as an example of what should appear if there is a successful connection:
![image](./pictures/runner.png)

# Secrets Per Repository

Official documentation for storing secrets can be found [here](https://docs.drone.io/secret/repository/). The steps below may also help:

> 1. Click on the desired repository, click on the `Settings` tab, and on the left-hand side choose `Secrets` as shown below:
![image](./pictures/drone-secrets.png) 
> 2. Click `NEW SECRET` and enter the key and value pair.
> 3. In your `drone.yaml` file, you can then access this secret as shown below:
```yaml
  steps:
  - name: build
  image: alpine
  environment:
    YOUR_ENVIRONMENT_VARIABLE:
      from_secret: my_secret_test
```
Note: The ephemeral Docker container which spins up when you run the pipeline will contain the environment variable, as shown below:

![image](./pictures/printenv.png)

However, in the console logs for the build, only asterisks (`******`) will appear if you try to print out this environment variable.

# Kubernetes Installation

This cluster will consist of two nodes. One node will be the master node, and the other one the agent node. 

## Commands to run on both nodes.

### Login as root user and disable swap
* Login as root user with `sudo su -`.
* Disable swap: `swapoff -a; sed -i '/swap/d' /etc/fstab`. 

### Update sysctl settings for Kubernetes networking
These commands allow IPtables to see bridged traffic.

```bash
cat >>/etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
sysctl --system
```

### Install Docker

```bash
apt update & sudo apt-upgrade -y
apt install docker.io
```

### Kubernetes Setup

Make sure these are all the same versions.

```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
apt update && sudo apt install -y kubeadm=1.25.4-00 kubelet=1.25.4-00 kubectl=1.25.4-00
```

## On Your Master Node Only

### Initialise Kubernetes Cluster

Run this command, making sure the network cidr is 10.244.0.0/16 (for flannel). Also note the ip address corresponds to the one found in the Vagrantfile: `kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.50.4 --ignore-preflight-errors=all`. 

After it has initialised, run `export KUBECONFIG=/etc/kubernetes/admin.conf`. 

### Deploy Flannel 

This will not work if you did not enter 10.244.0.0/16 as the network cidr address for the kubeadm init command.

Run `kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/v0.20.2/Documentation/kube-flannel.yml`. 

### Cluster Join Command 

Note down the output generated from this command for later use: `kubeadm token create --print-join-command`.

### Run the kubeadm join command on your agent nodes

This command can be found at the bottom of the output on the master node when you run the command `kubeadm token create --print-join-command`.

Run `kubectl get pods` in your agent nodes, and you should see an error. We will deal with this blocker.

### Dealing with the 8080 blocker

In your master node, go to `cd /etc/kubernetes/`. Copy admin.conf with `cat admin.conf`.

In your agent node, type `mkdir -p $HOME/.kube`.

Navigate to `cd ~/.kube` in your agent node.

In your agent node, type in `sudo nano config`, and paste all of the contents from admin.conf (which you should have copied from the master node admin.conf file).

Run `kubectl get nodes`, and you will see all of the nodes in the cluster.

