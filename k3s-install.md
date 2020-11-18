# Setting up a K3s cluster on WSL

## Install WSL and rebuild WSL kernel to support k3s

1. Install WSL2 following official docs

https://docs.microsoft.com/en-us/windows/wsl/wsl2-install

2. Compile Kernel
```bash
# get latest version of Kernel
git clone https://github.com/microsoft/WSL2-Linux-Kernel.git
cd WSL2-Linux-Kernel

# Install "compile kernel" tools
sudp apt update
sudo apt install build-essential flex bison libssl-dev libelf-dev

# Setup kernel config
cp Microsoft/config-wsl .config
cat >> .config << EOF
CONFIG_BRIDGE_NETFILTER=y
CONFIG_NETFILTER_XT_MATCH_COMMENT=y
CONFIG_NETFILTER_XT_MATCH_MULTIPORT=y
CONFIG_NETFILTER_XT_MATCH_OWNER=y
CONFIG_NETFILTER_XT_MATCH_PHYSDEV=y
CONFIG_VXLAN=y
CONFIG_GENEVE=y
EOF
make oldconfig

# Compile
make -j $(nproc)
```

## Update WSL kernel

1. Copy `arch/x86/boot/bzImage` to somewhere outside WSL2 (`/mnt/c/Users/${USER}/Desktop`)
2. Shutdown WSL2 `wsl --shutdown`
3. Make a copy of `c:\Windows\System32\lxss\tools\kernel` to `c:\Windows\System32\lxss\tools\kernel.bak.yyyymmdd` (the original kernel is still in another location)
4. Copy bzImage to `c:\Windows\System32\lxss\tools\kernel`. Might need to copy through explorer.exe because of permissions issues.

## Install K3s on WSL

You can't install k3s using the curl script because there is no supervisor (systemd or openrc) in WSL2.

1. Download k3s binary from https://github.com/rancher/k3s/releases/latest
```bash
# wget https://github.com/rancher/k3s/releases/download/v1.19.3%2Bk3s2/k3s
wget https://github.com/rancher/k3s/releases/download/v1.19.3%2Bk3s3/k3s
```
2. `chmod +x k3s`
3. Run k3s `sudo ./k3s server`

## Setup access to k3s
1. Copy `/etc/rancher/k3s/k3s.yaml` to `$HOME/.kube/config`.
```bash
mkdir -p $HOME/.kube/
sudo cp /etc/rancher/k3s/k3s.yaml $HOME/.kube/config
sudo chown $USER config
```
2. Get IP of the your WSL2 instance.
```bash
ip addr show dev eth0 | grep "inet " | awk '{print $2}' | cut -d'/' -f1
```
3. Edit the kubeconfig file and change the server URL from `https://<127_0_0_1_or_previous_ip>:6443` to the IP returned by the previous command.

4. (Optional) Copy kubconfig file from WSL `$HOME/.kube/config` to your home in Windows `%HOME%\.kube\config`:
```bash
WINHOME=`wslpath "$(wslvar USERPROFILE)"`
cp ~/.kube/config $WINHOME/.kube/
```
5. Run `kubectl`.
```bash
kubectl get nodes
```
> For installing kubectl follow the [installation guide](https://kubernetes.io/docs/tasks/tools/install-kubectl/).
## Credits
Adapted from [K3s on WSL: Quick Start Guide](https://gist.github.com/ibuildthecloud/1b7d6940552ada6d37f54c71a89f7d00).
