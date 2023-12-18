# K3s

## Node Requirements

Make sure that all nodes meet the [K3s node requirements](https://docs.k3s.io/installation/requirements).

### Hardware

Nodes should have:
- 2 cores
- 1 GB of RAM

### Networking Requirements

Per the list of [K3s requirements](https://docs.k3s.io/installation/requirements?os=debian#networking),
it seems like port `6443` may be the only inbound port that must be open
both for servers and agents.

Perform the steps in this section as `root`.

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

Open port 6443 with:

```bash
iptables -A INPUT -p tcp --dport 6443 -j ACCEPT
```

To make sure the rules persist, run:

```bash
apt install iptables-persistent
```

During the `iptables-persistent` installation you will be asked if you would like to
save the current set of rules to `/etc/iptables/rules.v4`.
Agree to do so.

## Installing the server

Following along with the instructions given [here](https://docs.k3s.io/installation/configuration#putting-it-all-together):

Create a config.yaml file at `/etc/rancher/k3s/config.yaml`, containing:

```yaml
token: "secret"
debug: true
```

Run the K3s installer script with:

```bash
curl -sfL https://get.k3s.io \
    | K3S_KUBECONFIG_MODE="644" \
      INSTALL_K3S_EXEC="server" \
      sh -s - --flannel-backend none
```

This creates a server with:

- A `kubeconfig` file with permissions 644
- [Flannel](https://docs.k3s.io/installation/network-options#flannel-options) backend set to none
- The token set to `secret`
- Debug logging enabled

## Installing the agent

TODO: Update to move token to file

```bash
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="agent --server https://k3s.example.com --token mypassword" sh -s -
```
