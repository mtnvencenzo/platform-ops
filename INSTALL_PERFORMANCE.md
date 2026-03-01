# Performance Tune Host Machine
[<< back](INSTALL.md)

Use the following settings to allow for heavy workloads on you host machine

## Setup System Limits
### Step 1: Ensure Inotify values are sufficent

First see what values are on the system. 
``` shell
sysctl fs.inotify
#fs.inotify.max_queued_events = 16384
#fs.inotify.max_user_instances = 65536
#fs.inotify.max_user_watches = 1048576
```

### Step 2: Append settings to /etc/sysctl.d/99-k3d-heavy-workloads.conf
``` shell
sudo tee /etc/sysctl.d/99-k3d-heavy-workloads.conf <<EOF
vm.max_map_count=262144
fs.inotify.max_user_instances=65536
EOF

# Apply the changes immediately without rebooting
sudo sysctl --system 

# verify
cat /proc/sys/vm/max_map_count
cat /proc/sys/fs/inotify/max_user_instances
cat /etc/sysctl.d/99-k3d-heavy-workloads.conf
```

### Step 3: Add file descriptor limits to /etc/security/limits.conf
``` shell
sudo tee -a /etc/security/limits.conf <<EOF
* soft nofile 65536
* hard nofile 65536
EOF

# verify
ulimit -n
ulimit -Sn  # View current Soft limit
ulimit -Hn  # View current Hard limit
```

## Setup Docker Data Directory 
Move dockers data directory to a separate SSD Drive for performance


### Step 1: Stop Docker

``` shell
sudo systemctl stop docker.service && sudo systemctl stop docker.socket
```

### Step 2: Create the data directory

``` shell
sudo mkdir -p /opt/docker
```

### Step 3: Configure docker daemon

``` shell
sudo tee -a /etc/docker/daemon.json <<'EOF'
{
  "data-root": "/opt/docker",
  "log-driver": "json-file",
  "log-opts": { "max-size": "10m", "max-file": "3" }
}
EOF
```

If you setup gpu support the end result should be similar to:

``` json
You should see output similar to:

```json
{
  "default-runtime": "nvidia",
  "runtimes": {
    "nvidia": {
      "path": "nvidia-container-runtime",
      "runtimeArgs": []
    }
  },
  "data-root": "/opt/docker",
  "log-driver": "json-file",
  "log-opts": { "max-size": "10m", "max-file": "3" }
}
```


### Step 4: Migrate data to new directory

``` shell
sudo rsync -aP /var/lib/docker/ /opt/docker/
```


### Step 5: Start docker

``` shell
sudo systemctl start docker
```