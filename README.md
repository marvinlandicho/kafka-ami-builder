# Apache Kafka AMI Builder

Packer template to create an AMI with Apache Zookeeper installed.

All required initialization scripts, packages and configurations are included in this script.
This can be used to instantiate a zookeeper server in both standalone or cluster mode.

### Prerequisites

Install Packer and AWS Command Line Interface tools on your local machine. I am using Mac
so most of the commands and instruction here should work in a Mac.

Building the base image requires the id of the latest Debian AMI files
for the region where you wish to build the AMI.

### Running packer

Below options are required to build the image. Check the json file
for other options that can be supplied.

```
Usage:
  packer build \
    -var 'aws_access_key=AWS_ACCESS_KEY' \
    -var 'aws_secret_key=<AWS_SECRET_KEY>' \
    -var 'aws_region=<AWS_REGION>' \
    -var 'zookeeper_version=<ZOOKEEPER_VERSION>' \
    [-var 'option=value'] \
    kafka.json
```

#### Script Options

- `aws_access_key` - *[required]* The AWS access key.
- `aws_ami_name` - The AMI name (default value: "kafka").
- `aws_ami_name_prefix` - Prefix for the AMI name (default value: "").
- `aws_instance_type` - The instance type to use for the build (default value: "t2.micro").
- `aws_region` - *[required]* The regions were the build will be performed.
- `aws_secret_key` - *[required]* The AWS secret key.
- `kafka_scala_version` - Kafka Scala version (default value: "2.11").
- `kafka_version` - *[required]* Kafka version.
- `system_locale` - Locale for the system (default value: "en_US").

#### Configuration Script

The script can and should be used to set some of the kafkfa options as well
as setting the kafka service to start at boot. The same script can be called
by ansible to start and configure the kafka

```
Usage: kafka_config [options]
```

##### Options

* `-a <ADDRESS>` - Sets the Kafka broker advertised address (default value is 'localhost').
* `-D` - Disables the Kafka service from start at boot time.
* `-E` - Enables the Kafka service to start at boot time.
* `-i <ID>` - Sets the Kafka broker ID (default value is '0').
* `-m <MEMORY>` - Sets Kafka maximum heap size. Values should be provided following the same Java heap nomenclature.
* `-S` - Starts the Kafka service after performing the required configurations (if any given).
* `-W <SECONDS>` - Waits the specified amount of seconds before starting the Kafka service (default value is '0').
* `-z <ENDPOINT>` - Sets a Zookeeper server endpoint to be used by the Kafka broker (defaut value is 'localhost:2181'). Several Zookeeper endpoints can be set by either using extra `-z` options or if separated with a comma on the same `-z` option.

#### Configuring a Kafka Broker


Start a single node kafka and zookeeper:

```
kafka_config -a kafka01.mydomain.tld -E -S -i 1 -z zookeeper01.mydomain.tld:2181
```

Start a multi-node kafka and zookeeper cluster:

```
kafka_config -a kafka01.mydomain.tld -a kafka02.mydomain.tld -a kafka03.mydomain.tld -E -S -i 1 -z zookeeper01.mydomain.tld:2181 -i 2 -z zookeeper02.mydomain.tld:2181 -i 3 -z zookeeper02.mydomain.tld:2181
```

#### Required Ports in Sec Group

| Service      | Port      | Protocol |
|--------------|:---------:|:--------:|
| SSH          | 22        |    TCP   |
| Zookeeper    | 2181      |    TCP   |
| Zookeeper    | 2888:3888 |    TCP   |
| Kafka Broker | 9092      |    TCP   |

