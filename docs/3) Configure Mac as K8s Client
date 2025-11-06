# Overview
To make running commands on the control plane easier, your laptop can be configured as a remote Kubernetes client so you can directly run kubectl on your laptop without needing to SSH.

### Inspect control plane's kubeconfig
```bash
sudo cat /etc/kubernetes/admin.conf
```

### Create a kube directory (if done doesn't already exist) and create/update the config
```bash
mkdir -p ~/.kube
nano ~/.kube/config
```
Paste the contents into the config. ***Update the server endpoint*** to the control plane node's reachable IP from the Mac.

Save it, and then try running `kubectl get nodes` on your Mac. This should work.