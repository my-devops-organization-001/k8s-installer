The connection to the server 192.168.112.134:6443 was refused - did you specify the right host or port?


kubelet is not runing
Run this to fix
sudo -i
sudo export KUBECONFIG="/home/adeyori/.kube/config
swapoff -a
exit
strace -eopenat kubectl version






ping 192.168.112.134
telnet 192.168.112.134 6443
nc -vz 192.168.112.134 6443
kubectl config view
export KUBECONFIG="/home/adeyori/.kube/config"
write ```export KUBECONFIG="/home/adeyori/.kube/config"``` into .bashrc
write ```sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
``` into .basrc
kubectl --kubeconfig="/home/adeyori/.kube" get nodes



E0921 07:07:15.466246    8507 memcache.go:265] couldn't get current server API group list: Get "http://localhost:8080/api?timeout=32s": dial tcp 127.0.0.1:8080: connect: connection refused

```
kubectl config view
apiVersion: v1
clusters: null
contexts: null
current-context: ""
kind: Config
preferences: {}
users: null
kubelet can not see the config file, kubele can not start, kubelet cannot start containerd

echo $KUBECONFIG empty
cat .kube/config is the file that contain the configuration 
openssl x509 -in /var/lib/kubelet/pki/kubelet.crt -text -noout

```


On worker nodes, you do not need to copy admin.conf or set up the .kube directory since the worker nodes don't directly manage the cluster. Worker nodes only need to communicate with the control plane through the kubelet and container runtime services.


df -h
sudo mount <partition> <mount_point>
df -h | grep -E 'Filesystem|100%'
sudo crictl info | grep -i 'RootDir'

kubeadm token create --print-join-command on the control plane




[ERROR FileAvailable--etc-kubernetes-kubelet.conf]: /etc/kubernetes/kubelet.conf already exists
	[ERROR Port-10250]: Port 10250 is in use
	[ERROR FileAvailable--etc-kubernetes-pki-ca.crt]: /etc/kubernetes/pki/ca.crt already exists
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher


sudo lsof -i :10250

sudo kubeadm reset




journalctl -u kubelet -n 100 --no-pager

{
	ip -brief address
ip route
cat /etc/resolv.conf
ping -4 1.1.1.1
curl -4I http://archive.ubuntu.com/ubuntu/
}



ping -c 2 1.1.1.1                                           # internet by IP address
curl -4 -sS -I --max-time 10 https://pkgs.k8s.io | head -1  # HTTPS (what Kubernetes needs)
curl -4 -sS -I --max-time 10 http://archive.ubuntu.com/ubuntu/ | head -1   # HTTP (what apt used)
ip link show eth0 | grep -o 'mtu [0-9]*'