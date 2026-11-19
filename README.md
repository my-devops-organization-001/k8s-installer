# What is this this
- These collection of script install and setup kubernetes with kubeadm
# Usage
- Before running ther script, ensure you set the version of the component to be install
in ["04-runc-cni-conterd-install"](04-runc-cni-containerd-install)
```bash
./install
```


# This is a set of script that automate the deployment kubernetes using kubeadm
- Start  install script and follow the prompt
- once a master has been initialized, other once will join as worker







Install kubectl
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl


Services


# /lib/systemd/system/mongo.service
# nginx signals reference doc:
# http://nginx.org/en/docs/control.html
#
[Unit]
Description=A high performance web server and a reverse proxy server
Documentation=man:nginx(8)
After=network.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t -q -g 'daemon on; master_process on;'
ExecStart=/usr/sbin/nginx -g 'daemon on; master_process on;'
ExecReload=/usr/sbin/nginx -g 'daemon on; master_process on;' -s reload
ExecStop=-/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/nginx.pid
TimeoutStopSec=5
KillMode=mixed

[Install]
WantedBy=multi-user.target


Here's how to adapt the systemd service unit file you provided for Nginx to work for MongoDB instead. This example assumes you have MongoDB installed in a standard way and uses common paths and settings for MongoDB services on a Linux system with systemd.

### MongoDB Systemd Service Unit File

```ini
[Unit]
Description=MongoDB Database Server
Documentation=https://docs.mongodb.com/manual
After=network.target

[Service]
User=mongodb
Group=mongodb
Type=forking
PIDFile=/var/run/mongodb/mongod.pid
ExecStartPre=/usr/bin/mongod --quiet --config /etc/mongod.conf --fork
ExecStart=/usr/bin/mongod --config /etc/mongod.conf --fork
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID
TimeoutStopSec=5
KillMode=process

[Install]
WantedBy=multi-user.target
```

### Explanation of Changes:

1. **[Unit] Section**:
    - **Description**: Changed from Nginx to MongoDB.
    - **Documentation**: Updated to point to MongoDB's official documentation.

2. **[Service] Section**:
    - **User** and **Group**: Specifies that MongoDB should run as the `mongodb` user and group. These should be set to whatever user/group your MongoDB installation uses.
    - **Type**: Changed to `forking`, as MongoDB's server process is a long-running daemon that forks when started.
    - **PIDFile**: The path to MongoDB's PID file, typically found at `/var/run/mongodb/mongod.pid` or similar. You should adjust this path based on your specific installation.
    - **ExecStartPre**: Removed the Nginx test command and replaced it with a pre-start command for MongoDB. MongoDB doesn’t have a direct flag equivalent to Nginx’s config test, so this simply starts MongoDB in a forking mode with the `--fork` flag.
    - **ExecStart**: The main command to start MongoDB with the desired configuration file.
    - **ExecReload**: Sends a hangup signal to reload the configuration without downtime.
    - **ExecStop**: Sends a terminate signal to gracefully stop MongoDB.
    - **TimeoutStopSec**: Time to wait for the service to stop before sending a SIGKILL.
    - **KillMode**: Defines how systemd will kill the service’s processes. Set to `process` to only kill the main process.

3. **[Install] Section**:
    - **WantedBy**: Ensures the MongoDB service is started at the appropriate runlevels.

### Notes:

- **Configuration File**: The `/etc/mongod.conf` path in `ExecStartPre` and `ExecStart` should point to the actual MongoDB configuration file on your system.
- **Permissions**: Make sure the `mongodb` user and group exist and have appropriate permissions on MongoDB's data and log directories.
- **Paths**: Adjust paths for the executable and PID file as needed depending on how MongoDB was installed on your system (e.g., from a package manager or manually compiled).

Before deploying this service file, ensure all paths and user settings are correct for your specific MongoDB setup. Once the service file is in place, you can enable and start MongoDB using systemd:

```bash
sudo systemctl enable mongod
sudo systemctl start mongod
```

Check the status of your MongoDB service with:

```bash
sudo systemctl status mongod
```

This setup should ensure that MongoDB starts automatically at boot and can be managed using standard systemd commands.

/192.168.112.134:6443/

Manifest --> object --> key-value pair



To determine if your container runtime is using `systemd` as the cgroup driver, you can follow these steps:

1. **Check the Container Runtime Configuration:**
   - For **Docker**, check the daemon configuration file, usually located at `/etc/docker/daemon.json`. Look for the `cgroup-driver` setting:
     ```json
     {
       "cgroup-driver": "systemd"
     }
     ```

   - For **containerd**, check the configuration file typically found at `/etc/containerd/config.toml`. Look for the `[plugins."io.containerd.grpc.v1.cri".containerd]` section and the `cgroup_driver` setting:
     ```toml
     [plugins."io.containerd.grpc.v1.cri".containerd]
       ...
       cgroup_driver = "systemd"
     ```

   - For **CRI-O**, check the configuration file at `/etc/crio/crio.conf` and look for the `cgroup_manager` setting:
     ```ini
     [crio]
     cgroup_manager = "systemd"
     ```

2. **Verify the Kubelet Configuration:**
   - Check the Kubelet service configuration, which might be located in the systemd service file (often `/etc/systemd/system/kubelet.service` or similar). Look for the `--cgroup-driver` flag:
     ```
     --cgroup-driver=systemd
     ```

3. **Check Runtime Information:**
   - You can also run the following command to check the cgroup driver in use by your kubelet and containers:
     ```bash
     ps aux | grep kubelet
     ```
   - This should show you the command line used to start the kubelet, including any cgroup driver flags.

4. **Using `kubectl`:**
   - If you have access to a Kubernetes cluster, you can also get information about the kubelet and its configuration:
     ```bash
     kubectl get nodes -o json | jq '.items[].status.nodeInfo'
     ```

By ensuring that both the kubelet and the container runtime are configured to use `systemd` as the cgroup driver, you can prevent any potential issues related to resource management in your Kubernetes cluster.


If you're using `containerd` as your container runtime, you generally do not need to install a separate Container Runtime Interface (CRI) layer, like `CRI-O`. `containerd` itself is a CRI-compatible runtime and can be used directly with Kubernetes.

### Key Points:

1. **Containerd as a CRI-Compatible Runtime**:
   - `containerd` provides an implementation of the CRI, so it can directly manage containers for Kubernetes without needing another layer.

2. **Installing CRI Plugin**:
   - You may need to ensure that the Kubernetes setup is configured to use the `containerd` CRI. This usually involves installing the `kubelet` and configuring it to use `containerd` as the container runtime.
   - You can typically set this up in the `kubelet` configuration file (or as command-line arguments).

3. **Configuration**:
   - Make sure that the `kubelet` is pointed to `containerd` by specifying the correct CRI socket in the kubelet configuration. This is usually done in the kubelet service configuration:
     ```
     --container-runtime-endpoint=/run/containerd/containerd.sock
     ```

4. **Using with Kubernetes**:
   - When deploying Kubernetes, if you use `kubeadm`, it automatically recognizes `containerd` as a valid runtime, as long as it's correctly configured.

### Summary

You do not need a separate CRI like `CRI-O` if you're using `containerd`, as it natively supports CRI. Just ensure that your `kubelet` is properly configured to interact with `containerd`. If you need help with specific configurations or setup, feel free to ask!

containerd config default |

sudo tee /etc/containerd/config.toml

sudo vim /etc/containerd/config.toml

sudo systemctl daemon-reload
sudo systemctl restart kubelet

sudo systemctl restart containerd

# node not ready
Based on the output, your root filesystem (`/dev/sda3`) has 24 GB total capacity, with 15 GB used and approximately 8.4 GB available (64% usage). This suggests that you have some free space available on your filesystem, but the Kubelet is reporting an "invalid capacity 0 on image filesystem." Here are some additional steps to troubleshoot this issue:

### 1. Verify Container Runtime Paths

Ensure that the paths where the container runtime (e.g., `containerd` or `docker`) stores images and containers are configured correctly. The default paths are:

- For **Containerd**: Typically `/var/lib/containerd`
- For **Docker**: Typically `/var/lib/docker`

You can check these directories to ensure they exist and have sufficient disk space:

```bash
df -h /var/lib/containerd
df -h /var/lib/docker
```

### 2. Check Containerd Configuration

Look at your containerd configuration file (`/etc/containerd/config.toml`) to confirm that the root directory for container storage is set correctly. The relevant section should look like this:

```toml
[plugins."io.containerd.grpc.v1.cri"]
  ...
  root = "/var/lib/containerd"
```

If it points to a different location, make sure that location has sufficient space and is correctly mounted.

### 3. Restart Containerd and Kubelet

After verifying and making any necessary changes, restart both the container runtime and Kubelet:

```bash
sudo systemctl restart containerd
sudo systemctl restart kubelet
```

### 4. Check Logs for Errors

After restarting, check the logs of the Kubelet and containerd for any errors:

```bash
journalctl -u kubelet -n 100 --no-pager
journalctl -u containerd -n 100 --no-pager
```

Look for any specific errors that might indicate what's going wrong.

### 5. Pod Events

Check if there are any events related to the pending pods:

```bash
kubectl describe pod <pod-name> -n kube-system
```

Replace `<pod-name>` with the names of the pending pods, such as `calico-kube-controllers` or `coredns`.

### 6. Disk Cleanup

If you're running low on space elsewhere or anticipate needing more space for container images, consider cleaning up unused images or containers:

For Docker:

```bash
docker system prune -a
```

For containerd, use `ctr` commands to manage images and containers.

### 7. Check for OverlayFS Issues

If you are using OverlayFS (common with container runtimes), ensure that the kernel supports it and that there are no issues with the underlying filesystem.

### 8. Reboot the Node

If all else fails, a system reboot might help resolve transient issues related to filesystems and mounted volumes.

### Final Steps

After performing these checks and potential fixes, monitor the status of your nodes and pods again:

```bash
kubectl get nodes
kubectl get pods -n kube-system
```

If issues persist, please provide any additional logs or error messages for further assistance!


