# K3s

## Node Requirements

Make sure that all nodes meet the [K3s node requirements](https://docs.k3s.io/installation/requirements).

### Hardware

Nodes should have:
- 2 cores
- 1 GB of RAM

### Networking Requirements

The following steps reflect the [K3s networking requirements](https://docs.k3s.io/installation/requirements?os=debian#networking).
Perform the steps in this section as `root` on both the server and agent nodes.

Install `iptables` with:

```bash
apt install iptables
```

Note: As root, I found that I needed to update my path to include `/usr/sbin`:

```bash
echo "export PATH=/usr/sbin:$PATH" > ~/.bashrc
source ~/.bashrc
```

Back up your current set of iptables rules with:

```bash
mkdir ~/backups
iptables-save > ~/backups/iptables_backup_$(date +"%Y%m%d_%H%M%S").rules
```

Open port 6443, the essential port used by Kubernetes, with:

```bash
iptables -A INPUT -p tcp --dport 6443 -j ACCEPT
```

Also open port 8472 which is required by the
[Flannel Container Network Interface (CNI) using the VXLAN backend](https://docs.k3s.io/installation/requirements#inbound-rules-for-k3s-server-nodes):

```bash
iptables -A INPUT -p udp --dport 8472 -j ACCEPT
```

Note that in this case the protocol is UDP.

To make sure the rules persist, run:

```bash
apt install iptables-persistent
```

During the `iptables-persistent` installation you will be asked if you would like to
save the current set of rules to `/etc/iptables/rules.v4`.
Agree to do so.

## Installing the server

Perform the steps in this section as `root`.

Following along with the instructions given [here](https://docs.k3s.io/installation/configuration#putting-it-all-together):

Create the `/etc/rancher/k3s` directory:

```bash
mkdir -p /etc/rancher/k3s
```

Create a config.yaml file at `/etc/rancher/k3s/config.yaml`, containing:

```yaml
token: "<k3s-token>"
debug: true
```

Run the K3s installer script with:

```bash
apt install -y curl
curl -sfL https://get.k3s.io \
    | K3S_KUBECONFIG_MODE="644" \
      INSTALL_K3S_EXEC="server" \
      sh -s - --flannel-backend vxlan
```

You should see output similar to the following:

```
[INFO]  Finding release for channel stable
[INFO]  Using v1.28.4+k3s2 as release
[INFO]  Downloading hash https://github.com/k3s-io/k3s/releases/download/v1.28.4+k3s2/sha256sum-amd64.txt
[INFO]  Downloading binary https://github.com/k3s-io/k3s/releases/download/v1.28.4+k3s2/k3s
[INFO]  Verifying binary download
[INFO]  Installing k3s to /usr/local/bin/k3s
[INFO]  Skipping installation of SELinux RPM
[INFO]  Creating /usr/local/bin/kubectl symlink to k3s
[INFO]  Creating /usr/local/bin/crictl symlink to k3s
[INFO]  Creating /usr/local/bin/ctr symlink to k3s
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s.service
[INFO]  systemd: Enabling k3s unit
Created symlink /etc/systemd/system/multi-user.target.wants/k3s.service → /etc/systemd/system/k3s.service.
[INFO]  systemd: Starting k3s
```

This creates a server with:

- A `kubeconfig` file with permissions 644 stored at `/etc/rancher/k3s/k3s.yaml`
- [Flannel](https://docs.k3s.io/installation/network-options#flannel-options) backend set to none
- The token set to `secret`
- Debug logging enabled

Furthermore, it installs a `k3s` binary at `/usr/local/bin/k3s`,
and creates a symbolic link at `/usr/local/bin/kubectl` that points to `k3s`.

## Getting information about the cluster

Run `kubectl get nodes` to ensure that the server node's status is `Ready`.
You should see something similar to the following:

```
NAME     STATUS   ROLES                  AGE   VERSION
totoro   Ready    control-plane,master   23h   v1.28.4+k3s2
```

Run `kubectl cluster-info` to gather additional information about the cluster.
You should see something similar to the following:

```
Kubernetes control plane is running at https://127.0.0.1:6443
CoreDNS is running at https://127.0.0.1:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://127.0.0.1:6443/api/v1/namespaces/kube-system/services/https:metrics-server:https/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

### Installing a dashboard

To install a dashboard, run:

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml
```

This will return output similar to the following:

```
namespace/kubernetes-dashboard created
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created
```

Next, create a proxy in order to access the dashboard
(which is normally only accessible from within the cluster):

```bash
kubectl proxy
```

You will now be able to visit
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login

You will need to be authorized to use this endpoint.

### Authenticating

https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/

https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
```

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```

```
kubectl -n kubernetes-dashboard create token admin-user
```


## Running a Sample App

https://www.digitalocean.com/community/tutorials/how-to-use-minikube-for-local-kubernetes-development-and-testing



```
kubectl create deployment web --image=gcr.io/google-samples/hello-app:1.0
```

```
kubectl expose deployment web --type=NodePort --port=8080
```

```
kubectl get service web
```

## Installing the agent

```bash
apt install -y curl
curl -sfL https://get.k3s.io \
    | INSTALL_K3S_EXEC="agent" \
      K3S_TOKEN="<k3s-token>" \
      sh -s - --server https://k3s.example.com
```

Note: Unlike with the server installation,
the agent installation seems to require supplying the token in the above command.
To avoid the secret being logged in the shell's history,
you could set the K3s token secret in a temporary environment variable.
