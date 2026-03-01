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

