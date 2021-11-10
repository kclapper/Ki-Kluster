# Ki-Kluster

Configuration and setup for Ki-Kluster based on K3S

## Things this repository does

These playbooks assume you have raspberry pi's setup with ubuntu server and that your DHCP service has reserved static ips for them.
They also assume you have ssh access to the pi's and have copied over your ssh-id. They also assume you have raspberry pi POE HATs installed 
on all of the pis and that you want the fan curves less agressive. 

### `rpiConfig.yml`

Handles shared configuration between all of the raspberry pis.
Things like:
- Changes the hostname to the hostname set in the inventory file (see config)
- Sets the ip to be static and disables dhcp
- Checks that cgroup configuration is correct (necessary for K3S)
- Adjusts POE HAT fan speed so it's less agressive. Fan should be off under light load.

### `k3s.yml`

Sets the raspberry pis up in a k3s cluster with one server node and the rest agent
nodes.
- Installs K3S server and agent based on quick start method (K3S_KUBECONFIG_MODE 644)
- Stores the value of the K3S_TOKEN found on the server node in 
/var/lib/rancher/k3s/server/node-token 
to be used when connecting the agent nodes. (this value becomes available in other plays as {{ hostvars[k3s_token_server]['k3s_token']['content'] | b64decode }})

### `k8sObjects.yml`

Takes care of cluster level configuration, setting up things like cert-manager and 
longhorn.

## Configuration

The server and agent nodes should be added under the 'inventory' file. There should only be one server and there can be multiple agents. Each node should be 
given a 'hostname' variable in the file like so:

    [server]
    10.0.8.1 hostname=serverhostname

    [agents]
    10.0.8.2 hostname=agenthostname1
    10.0.8.3 hostname=agenthostname2

Additionally, a `vars.yml` file should exist which contains the ip address of the server containing the K3S token. Like so:

    ---
    k3s_token_server: 10.0.8.1 

## Notes

`k3s.yml` is not configured for multiple K3S servers. It should also be noted that
the server install will most likely fail if the node is already setup as an agent 
and vice versa.

The `HA` directory contains materials for setting k3s up in high availability 
embedded mode. This repository was changed back to only setup one server after
the extent of performance problems with the embedded etcd on rpi was realized.

In `k8sObjects.yml`, Longhorn is setup specifically to kill terminating deployment pods in order to 
free up volumes to be reattached to new pods.