# Vagrantfile to setup Oracle Linux Cloud Native Environment on Oracle Linux 7
This Vagrantfile will deploy and configure the following components:

- One or more master nodes (one by default, 3 in HA mode)
- One or more worker nodes (2 by default)
- An optional operator node for the Oracle Linux Cloud Native Environment
Platform API Server and Platform CLI tool (default is to install these
components on the first master node)

If you enable multiple master nodes, an operator node is automatically deployed
to provide egress routing for the cluster.

All master and worker nodes will have the Oracle Linux Cloud Native
Environment Platform Agent installed and configured to communicate with the
Platform API Server on the operator node.

The installation includes the Kubernetes module for Oracle Linux Cloud
Native Environment which deploys Kubernetes 1.17.4 configured to use
the CRI-O runtime interface. Two runtime engines are installed, runc and
Kata Containers.

You may optionally enable the deployment of the Helm and Istio modules. Note
that enabling the Istio module will automatically enable the Helm module.

_Note:_ Kata Containers requires Intel hardware virtualization support and
will not work in a VirtualBox guest until nested virtualization support is
released for Intel CPUs.

## Prerequisites
1. Install [Oracle VM VirtualBox](https://www.virtualbox.org/wiki/Downloads)
1. Install [Vagrant](https://vagrantup.com/)
1. [vagrant-env](https://github.com/gosuri/vagrant-env) plugin is optional but
makes configuration much easier

## Quick start
1. Clone this repository `git clone https://github.com/oracle/vagrant-projects`
1. Change into the `vagrant-projects/OLCNE` folder
1. Run `vagrant up`

Your Oracle Linux Cloud Native Environment is ready!

From any master node (e.g. master1) you can check the status of the cluster (as
the `vagrant` user). E.g.:
- `kubectl cluster-info`
- `kubectl get nodes`
- `kubectl get pods --namespace=kube-system`

## About the Vagrantfile

The VMs communicate via a private network:

- Controller node (if any): 192.168.99.100
- Master node i: 192.168.99.(100+i) / master<em>i</em>.vagrant.vm
- Worker node i: 192.168.99.(110+i) / worker<em>i</em>.vagrant.vm

## Configuration
The Vagrantfile can be used _as-is_; there are a couple of parameters you
can set to tailor the installation to your needs.

### How to configure
There are several ways to set parameters:
1. Update the Vagrantfile. This is straightforward; the downside is that you
will loose changes when you update this repository.
1. Use environment variables. Might be difficult to remember the parameters
used when the box was instantiated.
1. Use the `.env`/`.env.local` files (requires
[vagrant-env](https://github.com/gosuri/vagrant-env) plugin). Configure
your cluster by editing the `.env` file; or better copy `.env` to `.env.local`
and edit the latter one, it won't be overridden when you update this repository
and it won't mark your git tree as changed (you won't accidentally commit your
local configuration!)

Parameters are considered in the following order (first one wins):
1. Environment variables
1. `.env.local` (if [vagrant-env](https://github.com/gosuri/vagrant-env) plugin
is installed)
1. `.env` (if [vagrant-env](https://github.com/gosuri/vagrant-env) plugin
is installed)
1. Vagrantfile definitions

### VM parameters
- `VERBOSE` (default: `false`): verbose output during VM deployment.
- `MEMORY` (default: 3072): all VMs are provisioned with 3GB memory.
- `VB_GROUP` (default: `OLCNE`): group all VirtualBox VMs under this label.

### Cluster parameters
- `STANDALONE_OPERATOR` (default: `false`): create a separate VM for the
operator node -- default is to install the operator components on the (first)
master node.
- `MULTI_MASTER` (default: `false`): multi-master setup. Deploy 3 masters in
HA mode.
__Note__: in multi-master mode, to circumvent a networking limitation, the
default route on master nodes needs to be on the private network interface
(`eth1`). To achieve this we use a non-master node as default gateway.
When `STANDALONE_OPERATOR` is `true`, we use the operator as gateway,
otherwise we take the first worker node (`worker1`). The gateway node must be
running or you masters will loose Internet connectivity!
- `NB_WORKERS` (default: 2): number of worker nodes to provision.
At least one worker node is required.
- `BIND_PROXY` (default: `false`): bind the kubectl proxy port (8001) from the
(first) master to the Vagrant host.
__Note__: you only need this if you want to expose the kubectl proxy to other
hosts in your network.

### Repositories
- `YUM_REPO` (default: none): additional yum repository to consider
(e.g. local repo)
- `OLCNE_DEV` (default: `false`): whether to enable the Oracle Linux Cloud
Native Environment developer channel.
- `REGISTRY_OLCNE` (default: `container-registry.oracle.com/olcne`): Container
registry for Oracle Linux Cloud Native Environment images.

For performance reasons, we recommend using the closest Oracle Container Registry mirror to your region. A list of available regions can be found on the [Regions and Availability Domains](https://docs.cloud.oracle.com/en-us/iaas/Content/General/Concepts/regions.htm) page of the Oracle Cloud Infrastructure documentation.

To specify an Oracle Container Registry mirror, either edit the Vagrantfile or install the vagrant-env plugin and create a .env.local file that specifies the mirror.

The following syntax can be used to specify a mirror:

- `container-registry-<region_name>.oracle.com`, e.g. `container-registry-sydney.oracle.com/olcne`
- `container-registry-<region_identifier>.oracle.com`, e.g. `container-registry-ap-sydney-1.oracle.com/olcne`
- `container-registry-<region_key>.oracle.com`, e.g. `container-registry-syd.oracle.com/olcne`

 All regions are available at <https://docs.cloud.oracle.com/en-us/iaas/Content/General/Concepts/regions.htm>


### Advanced Parameters
Danger zone!
Mainly used for development.

- The following parameters can be set to use specific component version:
`OLCNE_VERSION`, `K8S_VERSION`, `NGINX_IMAGE`.
- `NB_MASTERS` (default: none): override number of masters to deploy.

## Optional plugins
When installed, this Vagrantfile will make use of the following third party Vagrant plugins:
- [vagrant-env](https://github.com/gosuri/vagrant-env): loads environment
variables from .env files;
- [vagrant-hosts](https://github.com/oscar-stack/vagrant-hosts): maintains
/etc/hosts for the guest VMs;
- [vagrant-proxyconf](https://github.com/tmatilai/vagrant-proxyconf): set
proxies in the guest VMs if you need to access the Internet through proxy. See
plugin documentation for the configuration.

To intall Vagrant plugins run:
```
vagrant plugin install <name>...
```

## Product Documentation
* [Oracle Linux Cloud Native Environment: Getting Started](https://docs.oracle.com/en/operating-systems/olcne/start/index.html)
* [Oracle Linux Cloud Native Environment: Using Container Orchestration](https://docs.oracle.com/en/operating-systems/olcne/orchestration/index.html)
* [Oracle Linux Cloud Native Environment: Using Container Runtimes](https://docs.oracle.com/en/operating-systems/olcne/runtimes/index.html)

## Feedback
Please provide feedback of any kind via GitHub issues on this repository.

## Contributing
See [CONTRIBUTING](./CONTRIBUTING.md) for details.
