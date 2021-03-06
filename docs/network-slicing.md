# Network Slicing Engine

This external component provides a Network Slicing Engine (NSE). In the following fundamentals are described such as installing the NSE, configuring it and how to configure Network Slicing requirements.

In a nutshell this component ensures QoS configuration defined in the Descriptors provided by the NFVO.

The `network-slicing-engine` is implemented in java using the [spring.io] framework. It runs as an external component and communicates with the NFVO via Open Baton's SDK.

Additionally, the NSE uses the a plugin mechanism to allow whatever driver is needed to setup QoS. Currently, it supports only the neutron driver which allows to configure QoS in OpenStack directly. Hence, the NSE requires at least version Mitaka of OpenStack since it was recently introduced.

Before starting this component you have to do the configuration of the NSE that is described in the [next chapter](#manual-configuration-of-the-network-slicing-engine) followed by the guide of [how to start](#starting-the-network-slicing-engine) and [how to use](#how-to-use-the-network-slicing-engine) it.

# Technical Requirements
This section covers the requirements that must be met by the environment in order to satisfy the demands of the NSE:

* Installed and configured Open Baton NFVO/gVNFM (>=3.0.0)
* Installed and configured OpenStack (>=Mitaka) (for Mitaka check [here][os-neutron-mitaka-conf] or Newton [here][os-neutron-newton-conf]) with QoS APIs enabled

# How to install Network Slicing Engine
If you have the bootstrap procedure and selected the installation of the NSE component, you could skip this section, and move to the [How to use Network Slicing Engine](#how-to) one.


Otherwise, please continue with this section. Different options are available for the installation of the Network Slicing Engine. Either you use the fully automated bootstrap where all configurations are done automatically choosing between either the debian based installation or the source code one.
In case you don't use the bootstrap procedure, you can still install the NSE component individually either via apt-get or gradle.

## Installation via bootstrap

Using the bootstrap gives a fully automated standalone installation of the NS including installation and configuration.

The only thing to do is to execute the following command and follow the configuration process:

```bash
bash <(curl -fsSkl https://raw.githubusercontent.com/openbaton/network-slicing-engine/master/bootstrap)
```

Once you started the bootstrap you can choose between different options, such as installing this component via debian packages or from the source code (mainly for development)

## Installation via debian package

When using the debian package you need to add the apt-repository of Open Baton to your local environment with the following command if not yet done:

```bash
wget -O - http://get.openbaton.org/keys/public.gpg.key | apt-key add -
echo "deb http://get.openbaton.org/repos/apt/debian/ stable main" >> /etc/apt/sources.list
```

Once you added the repo to your environment you should update the list of repos by executing:

```bash
apt-get update
```

Now you can install the NSE by executing:

```bash
apt-get install openbaton-nse
```

## Installation from the source code

The latest stable version NSE can be cloned from this [repository][nse-repo] by executing the following command:

```bash
git clone https://github.com/openbaton/network-slicing-engine.git
```

Once this is done, go inside the cloned folder and make use of the provided script to compile the project as done below:

```bash
./network-slicing-engine.sh compile
```

# Manual configuration of the Network Slicing Engine

This chapter describes what needs to be done before starting the Network Slicing Engine. This includes the configuration file and properties, and also how to define QoS requirements in the descriptor.

## Configuration file
The configuration file must be copied to `etc/openbaton/network-slicing-engine.properties` by executing the following command from inside the repository folder:

```bash
cp etc/network-slicing-engine.properties /etc/openbaton/network-slicing-engine.properties
```

If done, check out the following chapter in order to understand the configuration parameters.

## Configuration properties

This chapter describes the parameters that must be considered for configuring the Network Slicing Engine.

| Params          				| Meaning       																|
| -------------   				| -------------																|
| logging.file					| location of the logging file |
| logging.level.*               | logging levels of the defined modules  |
| rabbitmq.host                 | host of RabbitMQ |
| rabbitmq.username             | username for authorizing towards RabbitMQ |
| rabbitmq.password             | password for authorizing towards RabbitMQ |
| nfvo.ip                       | IP of the NFVO |
| nfvo.port                     | Port of the NFVO |
| nfvo.username                 | username for authorizing towards NFVO |
| nfvo.password                 | password for authorizing towards NFVO |
| nfvo.project.name             | project used for registering for the events|

# Starting the Network Slicing Engine

How to start the NSE depends of the way you installed this component.

### Debian packages

If you installed the NSE with the debian packages you can start it with the following command:

```bash
openbaton-nse start
```

For stopping it you can just type:

```bash
openbaton-nse stop
```

### Source code

If you are using the source code you can start the NSE  easily by using the provided script with the following command:

```bash
./network-slicing-engine.sh start
```

Once the NSE is started, you can access the screen session by executing:

```bash
screen -r openbaton
```

For stopping you can use:
```bash
./network-slicing-engine.sh kill
```

**Note** Since the NSE subscribes to specific events towards the NFVO, you should take care about that the NFVO is already running when starting the NSE.

# <a name="how-to"></a>How to use Network Slicing Engine
The currently only supported driver is neutron, which will use the native QoS capabilities of Openstack Mitaka. To use it simply set ```nse.driver=neutron``` in the configuration file. To set QoS policies in your NSD specify the following QoS parameter in the virtual_link of your vnfd configuration.

In JSON:

```
  "virtual_link":[
    {
      "name":"NAME_OF_THE_NETWORK",
      "qos":[
        "maximum_bandwith:BRONZE"
      ]
    }
  ]
```
In YAML:

```yaml
mgmt:
  type: tosca.nodes.nfv.VL
  properties:
    vendor: Fokus
    qos:
      - minimum_bandwith:BRONZE

```


[nse-repo]: https://github.com/openbaton/network-slicing
[openbaton]: http://openbaton.org
[openbaton-doc]: http://openbaton.org/documentation
[spring.io]:https://spring.io/
[os-neutron-mitaka-conf]:http://docs.openstack.org/mitaka/networking-guide/config-qos.html
[os-neutron-newton-conf]:http://docs.openstack.org/newton/networking-guide/config-qos.html
