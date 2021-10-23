# Ki-Kluster

Configuration and setup for Ki-Kluster based on K3S

## Things this playbook does

This playbook assumes you have raspberry pi's setup with ubuntu server and that your DHCP service has reserved static ips for them.
It also assumes you have ssh access to the pi's and have copied over your ssh-id. It also assumes you have raspberry pi POE HATs installed 
on all of the pis and that you want the fan curves less agressive. 

It goes through and does a bunch of basic shared configuration then does role specific configuration (i.e K3S server vs agent config).

Shared Raspberry Pi Configuration:
- Changes the hostname to the hostname set in the inventory file (see config)
- Sets the ip to be static and disables dhcp
- Checks that cgroup configuration is correct (necessary for K3S)
- Adjusts POE HAT fan speed so it's less agressive. Fan should be off under light load.


K3S Server config:
- Installs K3S server based on quick start method (K3S_KUBECONFIG_MODE 644)
- Stores the value of the K3S_TOKEN found in /var/lib/rancher/k3s/server/node-token 
(this value becomes available in other plays as {{ hostvars[k3s_token_server]['k3s_token']['content'] | b64decode }})

K3S Agent config:
- Installs K3S agent based on quick start method using K3S_TOKEN found from server

## Configuration

The server and agent nodes should be added under the 'inventory' file. There should only be one server and there can be multiple agents. Each node should be 
given a 'hostname' variable in the file like so:

    [server]
    10.0.8.1 hostname=serverhostname

    [agent]
    10.0.8.2 hostname=agenthostname1
    10.0.8.3 hostname=agenthostname2

Additionally, a `vars.yml` file should exist which contains the ip address of the server containing the K3S token. Like so:

    ---
    k3s_token_server: 10.0.8.1 

## Notes

This playbook is not configured for multiple K3S servers. In the future, moving to high availability will require changing this playbook. 
It should also be noted that the server install will most likely fail if the node is already setup as an agent and vice versa.


## Longhorn

Longhorn was installed with a k8s-manifest:

    kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/v1.2.2/deploy/longhorn.yaml