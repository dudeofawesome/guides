```sh
sudo transactional-update pkg install \
  docker \
  EternalTerminal \
  fish \
  git \
  htop \
  k3s \
  lynx \
  most \
  ncdu \
  nvidia-glG05 \
  tmux \
  vim \
  wget \
  zfs
```

reboot


Set user's shell
```sh
chsh -s (which fish)
```


Enter transaction (and stay until specified!)
We use tmux here to ensure that even if our SSH session dies we don't loose our transaction.
```sh
tmux new -s transaction
sudo transactional-update shell
```

Enable passwordless root (used for Ansible)
```sh
visudo

# uncomment the passwordless sudo line
```

Add the user to some groups
```sh
usermod -aG wheel dudeofawesome
usermod -aG docker dudeofawesome
usermod -aG users dudeofawesome
```

```sh
systemctl enable et.service
systemctl enable docker.service
```

```sh
zypper addrepo https://download.nvidia.com/opensuse/tumbleweed NVIDIA
zypper addrepo https://download.opensuse.org/repositories/filesystems/openSUSE_Tumbleweed/filesystems.repo
```

```sh
exit # exit the transaction
```
reboot

```sh
sudo transactional-update pkg install \
  nvidia-glG05 \
  zfs
```
reboot

Enter transaction (and stay until specified!)
```sh
tmux new -s transaction
sudo transactional-update shell
```
