# Adding a backend

If you have an enterprise subscription (contact us), you can add your own hardware as backends.

## Install Ubuntu on a computer

It helps if the computer is a real computer, or a [cloud instance that works with Ignite](https://github.com/weaveworks/ignite/blob/main/docs/cloudprovider.md).

Ubuntu 20.04 is known to work.

## Access to the backend code

SSH into it, typically with the user `ubuntu`, then run the following commands on it:

```
ssh-keygen
```
Accept all the defaults (hit Enter until it stops asking questions).

```
cat ~/.ssh/id_rsa.pub
```

Send the server's public key to us and we'll add it to our GitLab instance.

```
git clone https://github.com/lukemarsden/luke-vimrc
luke-vimrc/setup.sh
```

Log out and log in again. This will change your `tmux` settings from the default just to wind you up (hint: use Ctrl+A).

## Ignite for running VMs

From [the ignite docs](https://github.com/weaveworks/ignite/blob/main/docs/installation.md).

```
sudo apt-get update
sudo apt-get install -y --no-install-recommends dmsetup openssh-client git binutils
```

We still use the docker backend, because reasons.
```
sudo apt-get install -y docker.io
```

```
export CNI_VERSION=v0.9.1
export ARCH=$([ $(uname -m) = "x86_64" ] && echo amd64 || echo arm64)
sudo mkdir -p /opt/cni/bin
curl -sSL https://github.com/containernetworking/plugins/releases/download/${CNI_VERSION}/cni-plugins-linux-${ARCH}-${CNI_VERSION}.tgz | sudo tar -xz -C /opt/cni/bin
```

Note we use an older version of ignite (we haven't updated our code to work with 0.9 yet).
```
export VERSION=v0.8.0
export GOARCH=$(go env GOARCH 2>/dev/null || echo "amd64")

for binary in ignite ignited; do
    echo "Installing ${binary}..."
    curl -sfLo ${binary} https://github.com/weaveworks/ignite/releases/download/${VERSION}/${binary}-${GOARCH}
    chmod +x ${binary}
    sudo mv ${binary} /usr/local/bin
done
```

## Get the backend code

Now for the good stuff.

```
sudo apt-get install -y software-properties-common gpg
sudo add-apt-repository ppa:longsleep/golang-backports
sudo apt-get update
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 52B59B1571A79DBC054901C0F6BC817356A3D45E
sudo apt-get install -y golang-go
```

```
sudo apt-get update
sudo apt install -y vim exuberant-ctags tmux
```

```
git clone git@gitlab.mlops.consulting:root/testfaster
```

```
cd testfaster/backend
```

```
cat <<'EOF' > .env
export DEBUG=1
export TARGET_API=https://testfaster.ci
export ACCESS_TOKEN=<backend-access-token>
export DOCKER_USERNAME=<docker-username>
export DOCKER_ACCESS_TOKEN=<docker-access-token>
export DISABLE_AUTOSCALING=1
EOF
```

Get your backend access token from us, and fill in a docker username and access token for a Docker Hub Pro account ($7/month) so that you can download images from Docker Hub.

## Testfaster agent

We'll handle distributing that docker token to your VMs, their k8s clusters and all service accounts that show up in any namespace in that Kubernetes cluster automatically for you.
We run an agent on each VM which does this for you.

That agent also rewrites `imagePullPolicy: Always` to `imagePullPolicy: IfNotPresent` so that starting VMs doesn't hammer Docker Hub when you've properly cached the images.

## Start backend

It's conventional to run all production software in a `tmux` session, right?

```
tmux
```

```
cd /home/ubuntu/testfaster/backend
```

```
sudo mkdir -p /var/lib/firecracker/vm
```

```
git pull; source .env; sudo -E go run testfaster_background.go 2>&1 |tee log 
```

Sometimes it may be useful to place `DEBUG_HTTP_ALL=1 DEBUG_HTTP=1` just before `sudo` in the above command.

## Monitoring

You can run a prometheus node exporter and we will monitor the disk, CPU and memory on the node for you.
We'll also be able to monitor VM queue length and various other interesting metrics in the future, and plan to expose these metrics in the Testfaster web interface.

```
sudo su
```
```
useradd --no-create-home --shell /bin/false node_exporter
```
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.1.2/node_exporter-1.1.2.linux-amd64.tar.gz
```
```
tar xvfz node_exporter-1.1.2.linux-amd64.tar.gz
cp node_exporter-1.1.2.linux-amd64/node_exporter /usr/local/bin
chown node_exporter:node_exporter /usr/local/bin/node_exporter
rm -rf node_exporter-1.1.2.linux-amd64*
```
```
cat <<'EOF' > /etc/systemd/system/node_exporter.service
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
EOF
```
```
systemctl daemon-reload
systemctl start node_exporter
```

Then tell us the IP address or hostname of your server and ask us to add it to our Prometheus instance.
