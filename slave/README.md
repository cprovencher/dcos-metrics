# Slave Modules
Monitoring component to be run against mesos-slaves. Contains an Isolator Module (tracks task bringup/shutdown) and a Hook which implements slaveExecutorEnvironmentDecorator (injects monitoring endpoints into Task environments), which must be enabled in the mesos slave via cmdline arguments.

## Prerequisites:

- CMake
- A nearby Mesos checkout and completed build: Set mesos_SOURCE_DIR and mesos_BUILD_DIR
- Protobuf (preferably what the Mesos build used)
- Boost ASIO (libasio-dev)

## Build instructions:

```
host:dcos-stats/slave$ ... build mesos ...
host:dcos-stats/slave$ sudo apt-get install cmake libasio-dev libboost-system-dev libgoogle-glog-dev
host:dcos-stats/slave$ mkdir build; cd build
host:dcos-stats/slave/build$ cmake -Dmesos_SOURCE_DIR=/path/to/mesos/ ..
host:dcos-stats/slave/build$ make -j4
```

## Install instructions

On a system running mesos-slave:

1. Copy dcos-stats/slave/build/modules.json and dcos-stats/slave/build/libstats-slave.so to the same directory.
2. Customize modules.json as needed (see parameters below).
3. Add line to /etc/mesos-slave/modules (create file if needed): "/path/to/modules.json"

## Customization

Available parameters for modules.json (see also params.hpp):

- "listen_host" (default "localhost"): Host to listen on for stats input from tasks.
- "listen_port_mode" (default "ephemeral"): Method to use for opening listen ports for stats input from tasks.
    - "ephemeral": Use OS-defined ephemeral ports for listen sockets. See /proc/sys/net/ipv4/ip_local_port_range.
    - "single": Use a single port across all tasks on the slave. Only advisable in ip-per-container environments. Requires the following additional arguments:
        - "listen_port": Port to listen on in "single" mode
    - "range": Use a defined range of ports for listening to tasks on the slave. Each task will use one port, monitoring data will be dropped if the number of tasks exceeds the number of ports. Requires the following additional arguments:
        - "listen_port_start": Start of range in "range" mode (inclusive)
        - "listen_port_end": End of range in "range" mode (inclusive)
- "dest_host" (default "statsd.monitoring.mesos"): Where to forward stats received from tasks.
- "dest_port" (default "8125"): Where to forward stats received from tasks.
- "annotations" (default "true"): Whether to use the [Datadog Tag statsd extension](http://docs.datadoghq.com/guides/dogstatsd/) ([wire format](https://github.com/DataDog/dogstatsd-python/blob/master/statsd.py#L178)) to annotate outgoing stats data with more information about the Mesos task.
- "chunking" (default "true"): Whether to group outgoing data into a smaller number of packets.
- "chunk_size_bytes" (default "512"): Preferred chunk size for outgoing UDP packets, when "chunking" is enabled.