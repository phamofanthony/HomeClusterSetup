# Objective
This doc outlines how I go about configuring my NAS and cluster with iSCSI to give my cluster storage. 

### Install SAN Manager on NAS
Done via the App Center.

### LUN/Target Creation
Create a LUN w/ thin volume

Create a target and map it to the LUN
- IQN/Name are identifiers and don't affect functinoality

Note down the NAS IP and target IQN for reference

### Configure iSCSI on all nodes
```bash
sudo apt update

sudo apt install -y open-iscsi

sudo systemctl enable iscsid --now

sudo iscsiadm -m discovery -t sendtargets -p [NAS_IP]

sudo iscsiadm -m node --login

sudo iscsiadm -m node -T [TARGET_IQN] --op update -n node.startup -v automatic
```

