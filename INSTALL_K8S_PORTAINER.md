# Install Portainer
[<< back](INSTALL.md)

## Install Portainer into the Cluster
To expose via Load Balancer, use the following command to provision Portainer at an assigned Load Balancer IP on port 9000 for HTTP and 9443 for HTTPS:
``` shell
kubectl apply -n portainer -f https://downloads.portainer.io/ce-lts/portainer-lb.yaml
```

### Restart Portainer
For some reason a fresh install requires a restart 'for security reasons'

``` shell
kubectl -n portainer rollout restart deployment.apps/portainer
```

### Setup account for the first time
Make sure to setup the account right way.  If not done within five minutes you'll have to delete the pod and start over.

When you go to the initial login screen it will ask for the setup token.  The token is in the logs
``` bash
# First find the name of the portainer pod
kube get pods -n portainer

# Read the logs using the pod name
kubectl logs portainer-54cd5d88bf-8bbp8 -n portainer

# Will look something like this:
# ==========================
#
# setup_token=111f23e187acub23448d766573147488a45f33u8b0f46adbf987617072d278e3
```

