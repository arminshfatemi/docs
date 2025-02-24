---
title: Networking overview
description: How networking works from the container's point of view
keywords: networking, container, standalone
aliases:
- /articles/networking/
- /config/containers/container-networking/
- /engine/userguide/networking/
- /engine/userguide/networking/configure-dns/
- /engine/userguide/networking/default_network/binding/
- /engine/userguide/networking/default_network/configure-dns/
- /engine/userguide/networking/default_network/container-communication/
- /engine/userguide/networking/dockernetworks/
---

Container networking refers to the ability for containers to connect to and
communicate with each other, or to non-Docker workloads.

A container has no information about what kind of network it's attached to,
or whether their peers are also Docker workloads or not.
A container only sees a network interface with an IP address,
a gateway, a routing table, DNS services, and other networking details.
That is, unless the container uses the `none` network driver.

This page describes networking from the point of view of the container,
and the concepts around container networking.
This page doesn't describe OS-specific details about how Docker networks work.
For information about how Docker manipulates `iptables` rules on Linux,
see [Packet filtering and firewalls](packet-filtering-firewalls.md).

## Published ports

By default, when you create or run a container using `docker create` or `docker run`,
the container doesn't expose any of its ports to the outside world.
Use the `--publish` or `-p` flag to make a port available to services
outside of Docker.
This creates a firewall rule in the host,
mapping a container port to a port on the Docker host to the outside world.
Here are some examples:

| Flag value                      | Description                                                                                                                                           |
| ------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| `-p 8080:80`                    | Map TCP port 80 in the container to port `8080` on the Docker host.                                                                                   |
| `-p 192.168.1.100:8080:80`      | Map TCP port 80 in the container to port `8080` on the Docker host for connections to host IP `192.168.1.100`.                                        |
| `-p 8080:80/udp`                | Map UDP port 80 in the container to port `8080` on the Docker host.                                                                                   |
| `-p 8080:80/tcp -p 8080:80/udp` | Map TCP port 80 in the container to TCP port `8080` on the Docker host, and map UDP port 80 in the container to UDP port `8080` on the Docker host. |

> **Important**
>
> Publishing container ports is insecure by default. Meaning, when you publish
> a container's ports it becomes available not only to the Docker host, but to
> the outside world as well.
>
> If you include the localhost IP address (`127.0.0.1`) with the publish flag,
> only the Docker host can access the published container port.
>
> ```console
> $ docker run -p 127.0.0.1:8080:80 nginx
> ```
>
> > **Warning**
> >
> > Hosts within the same L2 segment (for example, hosts connected to the same
> > network switch) can reach ports published to localhost.
> > For more information, see
> > [moby/moby#45610](https://github.com/moby/moby/issues/45610)
> { .warning }
{ .important }

If you want to make a container accessible to other containers,
it isn't necessary to publish the container's ports.
Inter-container communication is enabled by connecting the containers to the
same network, usually a [bridge network](./drivers/bridge.md).

## IP address and hostname

By default, the container gets an IP address for every Docker network it attaches to.
A container receives an IP address out of the IP subnet of the network.
The Docker daemon performs dynamic subnetting and IP address allocation for containers.
Each network also has a default subnet mask and gateway.

When you connect an existing container to a different network using `docker network connect`,
you can use the `--ip` or `--ip6` flags on that command
to specify the container's IP address on the additional network.

When a container starts, it can only attach to a single network, using the `--network` flag.
You can connect a running container to multiple networks using the `docker network connect` command.
When you start a container using the `--network` flag,
you can specify the IP address for the container on that network using the `--ip` or `--ip6` flags.

In the same way, a container's hostname defaults to be the container's ID in Docker.
You can override the hostname using `--hostname`.
When connecting to an existing network using `docker network connect`,
you can use the `--alias` flag to specify an additional network alias for the container on that network.

## DNS services

By default, containers inherit the DNS settings of the host,
as defined in the `/etc/resolv.conf` configuration file.
Containers that attach to the default `bridge` network receive a copy of this file.
Containers that attach to a
[custom network](network-tutorial-standalone.md#use-user-defined-bridge-networks)
use Docker's embedded DNS server.
The embedded DNS server forwards external DNS lookups to the DNS servers configured on the host.

You can configure DNS resolution on a per-container basis, using flags for the
`docker run` or `docker create` command used to start the container.
The following table describes the available `docker run` flags related to DNS
configuration.

| Flag           | Description                                                                                                                                                                                                                                                         |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `--dns`        | The IP address of a DNS server. To specify multiple DNS servers, use multiple `--dns` flags. If the container can't reach any of the IP addresses you specify, it uses Google's public DNS server at `8.8.8.8`. This allows containers to resolve internet domains. |
| `--dns-search` | A DNS search domain to search non-fully-qualified hostnames. To specify multiple DNS search prefixes, use multiple `--dns-search` flags.                                                                                                                            |
| `--dns-opt`    | A key-value pair representing a DNS option and its value. See your operating system's documentation for `resolv.conf` for valid options.                                                                                                                            |
| `--hostname`   | The hostname a container uses for itself. Defaults to the container's ID if not specified.                                                                                                                                                                          |

### Nameservers with IPv6 addresses

If the `/etc/resolv.conf` file on the host system contains one or more
nameserver entries with an IPv6 address, those nameserver entries get copied
over to `/etc/resolv.conf` in containers that you run.

For containers using musl libc (in other words, Alpine Linux), this results in
a race condition for hostname lookup. As a result, hostname resolution might
sporadically fail if the external IPv6 DNS server wins the race condition
against the embedded DNS server.

It's rare that the external DNS server is faster than the embedded one. But
things like garbage collection, or large numbers of concurrent DNS requests,
can result in a roundtrip to the external server being faster than local
resolution, on some occasions.

### Custom hosts

Custom hosts, defined in `/etc/hosts` on the host machine, aren't inherited by containers.
To pass additional hosts into container, refer to
[add entries to container hosts file](../engine/reference/commandline/run.md#add-host)
in the `docker run` reference documentation.

## Proxy server

If your container needs to use a proxy server, see
[Use a proxy server](proxy.md).