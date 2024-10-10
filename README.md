## About this container

[![Docker Pulls](https://img.shields.io/docker/pulls/Artoria2e5/rsntp.svg?logo=docker&label=pulls&style=for-the-badge&color=0099ff&logoColor=ffffff)](https://hub.docker.com/r/Artoria2e5/rsntp/)
[![Docker Stars](https://img.shields.io/docker/stars/Artoria2e5/rsntp.svg?logo=docker&label=stars&style=for-the-badge&color=0099ff&logoColor=ffffff)](https://hub.docker.com/r/Artoria2e5/rsntp/)
[![GitHub Stars](https://img.shields.io/github/stars/simonrupf/docker-chronyd.svg?logo=github&label=stars&style=for-the-badge&color=0099ff&logoColor=ffffff)](https://github.com/simonrupf/docker-chronyd)
[![Apache licensed](https://img.shields.io/badge/license-Apache-blue.svg?logo=apache&style=for-the-badge&color=0099ff&logoColor=ffffff)](https://raw.githubusercontent.com/simonrupf/docker-chronyd/master/LICENSE)

This container runs [chrony](https://chrony-project.org/) and [rsntp](https://github.com/mlichvar/rsntp) on [Alpine Linux](https://alpinelinux.org/).

* [chrony](https://chrony-project.org/) is a versatile implementation of the Network Time Protocol (NTP). It can synchronise the system clock with NTP servers, reference clocks (e.g. GPS receiver), and manual input using wristwatch and keyboard. It can also operate as an NTPv4 (RFC 5905) server and peer to provide a time service to other computers in the network.
* rsntp is a simple but high-performance NTP server. It is designed to run in multiple threads and serve the current system time, stealing the state from the "real" NTP server. It is useful for very high loads such as the cn.pool.ntp.org zone.

BECAUSE OF how rsntp works, this container MUST change your system time. If you are not comfortable with this, please do not use this container.

## Supported Architectures

Architectures officially supported by this Docker container. Simply pulling this container from [Docker Hub](https://hub.docker.com/r/Artoria2e5/rsntp/) should retrieve the correct image for your architecture.

![Linux x86-64](https://img.shields.io/badge/linux/amd64-green?style=flat-square)
![ARMv8 64-bit](https://img.shields.io/badge/linux/arm64-green?style=flat-square)
![IBM POWER8](https://img.shields.io/badge/linux/ppc64le-green?style=flat-square)
![IBM Z Systems](https://img.shields.io/badge/linux/s390x-green?style=flat-square)
![Linux x86/i686](https://img.shields.io/badge/linux/386-green?style=flat-squareg)
![ARMv7 32-bit](https://img.shields.io/badge/linux/arm/v7-green?style=flat-square)
![ARMv6 32-bit](https://img.shields.io/badge/linux/arm/v6-green?style=flat-square)


## How to Run this container

Before you go, tell conntrack to keep its hands off your thousands of connections:
```
iptables -t raw -A PREROUTING -p udp -m udp --dport 123 -j NOTRACK
iptables -t raw -A PREROUTING -p udp -m udp --dport 11123 -j NOTRACK
iptables -t raw -A OUTPUT -p udp -m udp --sport 123 -j NOTRACK
iptables -t raw -A OUTPUT -p udp -m udp --sport 11123 -j NOTRACK
ip6tables -t raw -A PREROUTING -p udp -m udp --dport 123 -j NOTRACK
ip6tables -t raw -A PREROUTING -p udp -m udp --dport 11123 -j NOTRACK
ip6tables -t raw -A OUTPUT -p udp -m udp --sport 123 -j NOTRACK
ip6tables -t raw -A OUTPUT -p udp -m udp --sport 11123 -j NOTRACK
```

### With the Docker CLI

Pull and run. But before you think "it's so simple, I should not read on",
please do at least read `run.sh` to figure out what to do to add stuff like PTP.

```bash-session
docker pull Artoria2e5/rsntp
docker run --name=ntp                           \
           --restart=always                     \
           --detach                             \
           --publish=123:123/udp                \
           --read-only                          \
           --tmpfs=/etc/chrony:rw,mode=1750     \
           --tmpfs=/run/chrony:rw,mode=1750     \
           --tmpfs=/var/lib/chrony:rw,mode=1750 \
           --cap-add=SYS_TIME                   \
           Artoria2e5/rsntp
```

### With Docker Compose

Using the docker-compose.yml file included in this git repo, you can build the container yourself (should you choose to).
*Note: this docker-compose files uses the `3.9` compose format, which requires Docker Engine release 19.03.0+

```
# run ntp
docker compose up -d ntp

# (optional) check the ntp logs
docker compose logs ntp
```


### With Docker Swarm

*(These instructions assume you already have a swarm)*

```
# deploy ntp stack to the swarm
docker stack deploy -c docker-compose.yml chronyd

# check that service is running
docker stack services chronyd

# (optional) view the ntp logs
docker service logs -f chronyd-ntp
```


### From a Local command line

Using the vars file in this git repo, you can update any of the variables to reflect your
environment. Once updated, simply execute the build then run scripts.

```
# build ntp
./build.sh

# run ntp
./run.sh
```


## Configure NTP Servers

By default, this container uses the [NTP pool's time servers](https://www.ntppool.org/en/). If you'd
like to use one or more different NTP server(s), you can pass this container an `NTP_SERVERS`
environment variable. This can be done by updating the [vars](vars), [docker-compose.yml](docker-compose.yml)
files or manually passing `--env=NTP_SERVERS="..."` to `docker run`.

Below are some examples of how to configure common NTP Servers.

Do note, to configure more than one server, you must use a comma delimited list WITHOUT spaces.

```
# (default) NTP pool
NTP_SERVERS="0.pool.ntp.org,1.pool.ntp.org,2.pool.ntp.org,3.pool.ntp.org"

# cloudflare (4 from pool)
NTP_SERVERS="time.cloudflare.com,time.cloudflare.com,time.cloudflare.com,time.cloudflare.com"

# google
NTP_SERVERS="time1.google.com,time2.google.com,time3.google.com,time4.google.com"

# alibaba
NTP_SERVERS="ntp1.aliyun.com,ntp2.aliyun.com,ntp3.aliyun.com,ntp4.aliyun.com"

# local (offline)
NTP_SERVERS="127.127.1.1"
```

(In real life you should *seriously* mix servers from different providers. Never put all your eggs in
one basket, especially when it comes to time. Get at least three from different providers -- the pool
automatically does this for you, but if you're using those corporate servers, you need to do it yourself.)

If you're interested in a public list of stratum 1 servers, you can have a look at the following lists.

 * https://www.advtimesync.com/docs/manual/stratum1.html (Do make sure to verify the ntp server is active
   as this list does appaer to have some no longer active servers.)
 * https://support.ntp.org/Servers/StratumOneTimeServers

It can also be the case that your use-case does not require a stratum 1 server -- most use-cases don't!

 * https://support.ntp.org/Servers/StratumTwoTimeServers

## RSNTP Options

RSNTP options are simple: you just set the number of threads, plus the nice level. We have those variables:

* `RSNTP_THREADS` - the number of threads to use. Default is `$(nproc)`.
  * `RSNTP_THREADS_4` - the number of threads for IPv4. Default is `max(1, RSNTP_THREADS * 2 / 3)`.
  * `RSNTP_THREADS_6` - the number of threads for IPv6. Default is `max(1, RSNTP_THREADS - RSNTP_THREADS_6)`.
  * We DO NOT check that the sum of these two is equal to `RSNTP_THREADS`, and it honestly doesn't matter, because
    only the `_4` and `_6` variables are used in the final invocation.
* `RSNTP_NICE` - the nice level to run at. Default is `5`, because things can get a bit busy and NTP is usually
  a side task. If you're running this on a dedicated machine, you might want to set this to `0`.

## Chronyd Options

### No Client Log (noclientlog)

This is optional and not enabled by default. If you provide the `NOCLIENTLOG=true` envivonrment variable,
chrony will be configured to:

> Specifies that client accesses are not to be logged. Normally they are logged, allowing statistics to
> be reported using the clients command in chronyc. This option also effectively disables server support
> for the NTP interleaved mode.

*This is not really useful, because the sole client should be rsntp -- localhost.*

## Logging

By default, this project logs informational messages to stdout, which can be helpful when running the
ntp service. If you'd like to change the level of log verbosity, pass the `LOG_LEVEL` environment
variable to the container, specifying the level (`#`) when you first start it. This option matches
the chrony `-L` option, which support the following levels can to specified: 0 (informational), 1
(warning), 2 (non-fatal error), and 3 (fatal error).

Feel free to check out the project documentation for more information at:

 * https://chrony-project.org/documentation.html


## Setting your timezone

By default the UTC timezone is used, however if you'd like to adjust your NTP server to be running in your
local timezone, all you need to do is provide a `TZ` environment variable following the standard TZ data format.
As an example, using `docker-compose.yaml`, that would look like this if you were located in Vancouver, Canada:

```yaml
  ...
  environment:
    - TZ=America/Vancouver
    ...
```


## Enable Network Time Security

If **all** the `NTP_SERVERS` you have configured support NTS (Network Time Security) you can pass the `ENABLE_NTS=true`
option to the container to enable it. As an example, using `docker-compose.yaml`, that would look like this:

```yaml
  ...
  environment:
    - NTP_SERVER=time.cloudflare.com,time.cloudflare.com,time.cloudflare.com,time.cloudflare.com
    - ENABLE_NTS=true
    ...
```

If any of the `NTP_SERVERS` you have configured does not support NTS, you will see a message like the
following during startup:

> NTS-KE session with 164.67.62.194:4460 (tick.ucla.edu) timed out


## Enable control of system clock

This option enables the control of the system clock.

By default, chronyd will not try to make any adjustments of the clock. It will assume the clock is free running
and still track its offset and frequency relative to the estimated true time. This allows chronyd to run without
the capability to adjust or set the system clock in order to operate as an NTP server.

Enabling the control requires granting SYS_TIME capability and a container run-time allowing that access:

```yaml
  ...
  cap_add:
    - SYS_TIME
  environment:
    - ENABLE_SYSCLK=true
    ...
```

## Enable the use of a PTP clock

If you have a `/dev/ptp0`, either a real hardware clock or virtual one provided by a VM host
you can enable the use of it by passing the device to the container. As an example,
using `docker-compose.yaml`, that would look like this:

```yaml
  ...
  devices:
    - /dev/ptp0:/dev/ptp0
```

This will allow chronyd to use the PTP clock as a reference clock. A virtual clock simply provides
the host's system time with great precision and stability; whether that time is accurate depends
on the host provider. In our experience, some VPS vendors give pretty good time (off by
milliseconds), while others are off by seconds.

For information on configuring the host to have a virtual PTP clock, see the following:

 * https://opensource.com/article/17/6/timekeeping-linux-vms


## Testing your NTP Container

From any machine that has `ntpdate` you can query your new NTP container with the follow
command:

```
ntpdate -q <DOCKER_HOST_IP>
```


Here is a sample output from my environment:

```bash-session
$ ntpdate -q 10.13.13.9
server 10.13.1.109, stratum 4, offset 0.000642, delay 0.02805
14 Mar 19:21:29 ntpdate[26834]: adjust time server 10.13.13.109 offset 0.000642 sec
```


If you see a message, like the following, it's likely the clock is not yet synchronized.
You should see this go away if you wait a bit longer and query again.
```bash-session
$ ntpdate -q 10.13.13.9
server 10.13.13.9, stratum 16, offset 0.005689, delay 0.02837
11 Dec 09:47:53 ntpdate[26030]: no server suitable for synchronization found
```

To see details on the ntp status of your container, you can check with the command below
on your docker host:
```bash-session
$ docker exec ntp chronyc tracking
Reference ID    : D8EF2300 (time1.google.com)
Stratum         : 2
Ref time (UTC)  : Sun Mar 15 04:33:30 2020
System time     : 0.000054161 seconds slow of NTP time
Last offset     : -0.000015060 seconds
RMS offset      : 0.000206534 seconds
Frequency       : 5.626 ppm fast
Residual freq   : -0.001 ppm
Skew            : 0.118 ppm
Root delay      : 0.022015510 seconds
Root dispersion : 0.001476757 seconds
Update interval : 1025.2 seconds
Leap status     : Normal
```


Here is how you can see a peer list to verify the state of each ntp source configured:
```bash-session
$ docker exec ntp chronyc sources
210 Number of sources = 2
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^+ time.cloudflare.com           3  10   377   404   -623us[ -623us] +/-   24ms
^* time1.google.com              1  10   377  1023   +259us[ +244us] +/-   11ms
```


Finally, if you'd like to see statistics about the collected measurements of each ntp
source configured:
```bash-session
$ docker exec ntp chronyc sourcestats
210 Number of sources = 2
Name/IP Address            NP  NR  Span  Frequency  Freq Skew  Offset  Std Dev
==============================================================================
time.cloudflare.com        35  18  139m     +0.014      0.141   -662us   530us
time1.google.com           33  13  128m     -0.007      0.138   +318us   460us
```

Like any host on your network, simply use your preferred ntp client to pull the time from
the running ntp container on your container host.

---
<a href="https://www.buymeacoffee.com/cturra" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/default-yellow.png" alt="Buy Me A Coffee" height="41" width="174"></a>
