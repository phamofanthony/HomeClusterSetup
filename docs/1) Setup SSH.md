# Objective
Now that I have my nodes running Ubuntu, I want to setup SSH so that I can connect to them remotely from my Macbook to run commands.

## Install OpenSSH Server on Ubuntu nodes
On the Ubuntu machine, Install openssh-server and enable SSH
```
sudo apt update
sudo apt install openssh-server -y
sudo systemctl enable ssh
sudo systemctl start ssh
sudo systemctl status ssh
```

## Create SSH key pair on Mac and copy it over to the Ubuntu nodes
On the Mac, create a keypair and copy it over
```
ssh-keygen -t ed25519 -C "myemail@gmaill.com"
ssh-copy-id username@node_ip_address
```

## Configure the SSH daemon to use key authentication
Open the SSH config
```
sudo nano /etc/ssh/sshd_config
```

Uncomment and set the following lines:
```
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
```

Restart SSH
```
sudo systemctl restart ssh
```

## Verify SSH connection works
Run the following command
```
ssh username@node_ip_address
```