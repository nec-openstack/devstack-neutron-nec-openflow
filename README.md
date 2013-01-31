DevStack for NEC OpenFlow plugin (Folsom)
=========================================

DevStack is a set of scripts and utilities to quickly deploy an OpenStack cloud.
This repository provides DevStack (Folsom release of OpenStack) with NEC OpenFlow pluign support.

# Basic usage

Prepare localrc (configuration file of DevStack).
* Sample configuration files are available in samples subdirectory.
  * samples/localrc, samples/local.sh : examples distributed with DevStack upstream
  * NEC OpenFlow plugin specific
    * samples/nec-openflow/localrc : localrc for All-in-One setup or Controller node in multi-node setup
    * samples/nec-openflow/localrc-hv : localrc for Compute node in multi-node setup
    * samples/common/01-proxy : Example of proxy settings
    * samples/common/02-repos : Example of local mirror
* For the detail how to customize localrc, please see below.

Next, run devstack

    $ ./stack.sh

# All-in-One (Single node)

In All-in-One mode all components of OpenStack and OpenFlow controller on a single machine.
If you would like to run them on multiple physical machines, please see **Multi-Node setup** section below.

## Configuration parameters

The sample localrc is available at samples/nec-openflow/localrc.

* **_PASSWORD** : Password for all components
* **OFC_HOST** : IP adddress of the OpenFlow controller which provides REST API (Sliceable Network Management API)
* **OFC_PORT** : TCP port of the OpenFlow controller for REST API
* **OFC_DRIVER** : Shortcut name or full class path of OpenFlow controller driver.
  * trema : Trema Sliceable Switch
  * pfc_v3 : ProgrammableFlow Controller V3.0
  * pfc_v4 : ProgrammableFlow Controller V4.0
* **USE_CUSTOM_OVS** : If True DevStack does not install Ubuntu package of Open vSwitch.
  It is useful if you want to use your special or customized version of Open vSwitch.
  You must install Open vSwitch manually **in advance**.

If you want to use OpenFlow controller running on a different host, configure OFC_HOST and OFC_PORT.

See [Devstack][devstack] for more information or read the codes :-)

## Start devstack

To start devstack, just type:

    $ ./stack.sh

## Stop devstack

To stop devstack:

    $ ./unstack.sh

# Multi-Node setup

## Network requirements

At the minimum setup, two networks are required.

* ***Control Plane Network*** :
  A network used for Quantum internal communication, communication among
  OpenStack components and OpenFlow control channel between OpenFlow controler and OpenFlow switch(es).
* ***Data Network*** : A network for user data traffic controlled by OpenFlow. If you don't have
  any hardware OpenFlow switch, you can use GRE tunnel option (GRE_REMOTE_IPS) to create OpenFlow Network
  over Control Plane Network (in this case only one network is required).

## Cluster Controller Node

### Configurations

The sample localrc is available at ***samples/nec-openflow/localrc***.

* **_PASSWORD** : Password for all components
* **OFC_HOST** : IP adddress of the OpenFlow controller which provides REST API (Sliceable Network Management API)
* **OFC_PORT** : TCP port of the OpenFlow controller for REST API
* **OFC_DRIVER** : Shortcut name or full class path of OpenFlow controller driver.
* **USE_CUSTOM_OVS** : If True DevStack does not install Ubuntu package of Open vSwitch.
* **OVS_INTERFACE**: Uncomment the line and set a physical network interface name
  (e.g., eth1) connected to an OpenFlow enabled network.
  The interface specified in OVS_INTERFACE will be attached to an Open vSwitch integration bridge ***br-int***
  (the bridge name is specified by OVS_BRIDGE variable). When you use tunneling like GRE
  to connect to other nodes, comment out this line (or set an empty value).
* **GRE_REMOTE_IPS**: Colon-separated list of remote IP adresses (e.g., 192.168.122.102:192.168.122.103).
  GRE tunnels from an Open vSwitch bridge (specified by OVS_BRIDGE) to each specified remote IP address
  will be created.

See [Devstack][devstack] for more information.

### Start the controller node

Start devstack:

    $ ./stack.sh

## Compute Node

On a compute node, nova-compute and quantum-plugin-agent runs.

### Configurations

The sample localrc is available at ***samples/nec-openflow/localrc-hv***. Copy it to localrc.

* **CC_HOST**: IP Address of the Cluster Controller on the control network
* **_PASSWORD** : Password for all components
* **OVS_INTERFACE**: A physical network interface name
  (e.g., eth1) connected to an OpenFlow enabled network.
  The interface specified in OVS_INTERFACE will be attached to an Open vSwitch bridge
  (named as OVS_BRIDGE) during the installation. When you use tunneling like GRE
  to connect to other nodes, comment out this line (or set an empty value).
* **GRE_REMOTE_IPS**: Remote IP adresses joined with colon
  (e.g., 192.168.122.102:192.168.122.103).
  GRE tunnels from an Open vSwitch bridge (named as OVS_BRIDGE) to
  the specified remote IP addresses will be configured during the installation.

See [Devstack][devstack] for more information.

### Start the compute node

    $ ./stack.sh

There are no order to start the controller node and compute nodes, but if you are not familiar with OpenStack,
we recommend to start the controller node first and start compute nodes after stack.sh on the controller finished.

[devstack]: http://devstack.org/
