# Chapter 17. Introduction to Networking

## Goal
Describe basic TCP/IP networking concepts and investigate the current network configuration and functionality on a server.

## Sections
- Networking Concepts (and Quiz)
- Validating Network Configuration (and Guided Exercise)

---

# 17.1. Networking Concepts

## Objectives
Describe the fundamental concepts of network addressing and routing for a server.

## The TCP/IP Network Model

The TCP/IP network model is a four-layer set of communication protocols that describes how data communications are packetized, addressed, transmitted, routed, and received between computers over a network. The protocol is specified by RFC 1122: Requirements for Internet Hosts - Communication Layers.

### The Four Layers of TCP/IP

#### Application Layer
- Each application has specifications for communication so that clients and servers can communicate across platforms
- **Common protocols include:**
  - **SSH** (remote access)
  - **HTTPS** (secure web)
  - **FTP** (file sharing)
  - **SMTP** (electronic mail delivery)

#### Transport Layer
- **TCP (Transmission Control Protocol)**: Provides reliable, connection-oriented communication method, ensuring data delivery and order
- **UDP (User Datagram Protocol)**: Connectionless datagram protocol, offering faster but less reliable communication

**Key concepts:**
- Application protocols use specific TCP or UDP ports to communicate
- Comprehensive list of well-known and registered ports: `/etc/services` file
- **Socket**: Combination of IP address and service port
- Every network packet includes source socket and destination socket

#### Internet Layer (Network Layer)
- Responsible for transmitting data packets from source host to destination host across interconnected networks
- **Key protocols**: IPv4 and IPv6
- Each host has unique IP address and prefix (e.g., `192.168.3.19/24`)
- **Routers**: Connect different networks and forward traffic between them

#### Link Layer (Media Access Layer)
- Provides connection to physical media
- **Common types:**
  - **Wired Ethernet (802.3)**
  - **Wireless Wi-Fi (802.11)**
- **MAC Address**: Media Access Control address (hardware address) identifies destination of packets on local network segment

### OSI vs TCP/IP Model Comparison

Although TCP/IP is the standard for networking, the Open Systems Interconnection (OSI) model remains a popular teaching reference due to its more granular structure.

**Common OSI references:**
- **"Layer 2"** for Ethernet
- **"Layer 3"** for IP
- **"Layer 4"** for TCP

## Network Interface Naming

Each network device in a system is identified and configured using its unique name. In Red Hat Enterprise Linux 7 and later, the system generates consistent network interface names that persist across reboots.

### Interface Name Structure

**Interface names start with type:**
- **`en`**: Ethernet interface names
- **`wl`**: WLAN interface names  
- **`ww`**: WWAN interface names

**Rest of interface name based on:**
- Information from server's firmware
- Location of device in PCI topology

### Naming Convention Examples

| Format | Description | Example |
|--------|-------------|---------|
| **`oN`** | Onboard device with unique index N | `eno1` (onboard Ethernet device 1) |
| **`sN`** | Device in PCI hotplug slot N | `ens3` (Ethernet card in PCI hotplug slot 3) |
| **`pMsN`** | PCI device on bus M in slot N | `enp2s3` |
| **`fN`** | Function N of multifunction device | `enp0s1f0` (function 0), `enp0s1f1` (function 1) |

**Example:** `wlp4s0` = WLAN card on PCI bus 4 in slot 0

### Benefits of Persistent Naming

- Network interface names stay the same even after adding/removing hardware
- Names generated based on unique hardware information
- More predictable than old `eth0`, `eth1`, `eth2` convention
- PCIe standard doesn't guarantee detection order, so persistent naming prevents unexpected changes

## IPv4 Networking

System administrators must have basic understanding of IPv4 networking to manage networking on servers. Although IPv6 has surpassed IPv4 usage in cellular networks, IPv4 remains the most common addressing scheme in enterprise networks.

### IPv4 Addresses

- **32-bit number** expressed as four 8-bit octets
- **Decimal format**: Values from 0 to 255, separated by dots
- **Two parts**: Network prefix and host identifier
- **Network prefix**: Identifies unique physical or virtual subnet
- **Host identifier**: Represents specific host on subnet

### Network Masks (Netmasks)

A **netmask** is a binary mask whose length indicates how many bits belong to the network prefix.

**Two notations:**
1. **CIDR notation**: `/24` (number of bits in binary mask)
2. **Decimal format**: `255.255.255.0` (four 8-bit octets)

### IPv4 Subnets

The number of available host addresses depends on network prefix size:

| Prefix | Host Bits | Possible Hosts | Example |
|--------|-----------|----------------|---------|
| `/24` | 8 bits | 254 hosts | `192.168.1.0/24` |
| `/16` | 16 bits | 65,534 hosts | `172.16.0.0/16` |
| `/8` | 24 bits | 16,777,214 hosts | `10.0.0.0/8` |

### Special Addresses in Subnets

- **Network address**: Lowest address (host identifier all zeros)
- **Broadcast address**: Highest address (host identifier all ones)
- **Gateway address**: Any unique host identifier (commonly first available)

### Example Network Calculations

#### Example 1: 192.168.1.107/24

| Component | Decimal | Binary |
|-----------|---------|--------|
| Host address | 192.168.1.107 | 11000000.10101000.00000001.01101011 |
| Netmask /24 | 255.255.255.0 | 11111111.11111111.11111111.00000000 |
| Network address | 192.168.1.0 | 11000000.10101000.00000001.00000000 |
| Host range | 192.168.1.1 - 192.168.1.254 | 00000001 to 11111110 |
| Broadcast address | 192.168.1.255 | 11000000.10101000.00000001.11111111 |

#### Example 2: 172.16.181.23/19

| Component | Decimal | Binary |
|-----------|---------|--------|
| Host address | 172.16.181.23 | 10101100.00010000.10110101.00010111 |
| Netmask /19 | 255.255.224.0 | 11111111.11111111.11100000.00000000 |
| Network address | 172.16.160.0 | 10101100.00010000.10100000.00000000 |
| Host range | 172.16.160.1 - 172.16.191.254 | 8,190 usable addresses |
| Broadcast address | 172.16.191.255 | 10101100.00010000.10111111.11111111 |

### Using ipcalc Command

Red Hat Enterprise Linux includes the `ipcalc` command-line tool for subnetting tasks:

```bash
user@host:~$ ipcalc 10.1.1.18/8
Address: 10.1.1.18
Network: 10.0.0.0/8
Netmask: 255.0.0.0 = 8
Broadcast: 10.255.255.255
Address space: Private Use
HostMin: 10.0.0.1
HostMax: 10.255.255.254
Hosts/Net: 16777214
```

## IPv4 Routing

Network packets move from host to host on a subnet and through routers from network to network. Each host uses a **routing table** to determine which network interface to use for sending packets to particular networks.

### Routing Table Entries

A routing table entry contains:
- **Destination network**
- **Network interface to use**
- **IP address of router** (if needed)

### Routing Logic

1. Host uses routing table entry that matches network prefix of destination address
2. If multiple entries are valid, use entry with **longer prefix**
3. If no specific match, route to **default gateway** (`0.0.0.0/0`)
4. Default route must point to router on local subnet

### Example Routing Table

| Destination | Interface | Router (if needed) |
|-------------|-----------|-------------------|
| 192.168.5.0/24 | enp0s1f0 | - |
| 192.168.6.0/24 | enp0s2f0 | - |
| 0.0.0.0/0 (default) | enp0s1f0 | 192.168.5.1 |

## IPv4 Address Configuration

### Dynamic Configuration (DHCP)

**DHCP (Dynamic Host Configuration Protocol)** process:
1. DHCP client broadcasts discovery request on local network
2. DHCP server responds with offer
3. Client requests and receives:
   - Unique IPv4 address
   - Subnet mask
   - Default gateway
   - DNS server addresses
4. Assignment valid for defined lease duration
5. Client must periodically renew lease

### Static Configuration

- Network settings read from local configuration files
- Settings must be appropriate for your subnet
- Coordinate with network administrator to avoid conflicts

---

# IPv6 Networking

IPv6 is widely deployed in enterprise networks and mobile communications. Many ISPs use IPv6 extensively for core infrastructure and dynamic address assignment. Designed to overcome IPv4 address exhaustion.

## IPv6 Addresses

- **128-bit number** shown as eight colon-separated groups of four hexadecimal digits
- Each hexadecimal digit (nibble) represents 4 bits
- Each quartet represents 16 bits

### IPv6 Address Format

**Full format:**
```
2001:0db8:0000:0010:0000:0000:0000:0001
```

**Compressed format (omit leading zeros):**
```
2001:db8:0:10:0:0:0:1
```

**Further compressed (:: for consecutive zeros):**
```
2001:db8:0:10::1
```

### IPv6 Address Writing Rules

1. **Suppress leading zeros** in groups (`0010` becomes `10`)
2. **Use double colon (::)** to shorten address as much as possible
3. **Double colon can appear only once** to avoid ambiguity
4. **Don't use :: for single group of zeros** (use `:0:` instead)
5. **Always use lowercase letters** (a-f) for hexadecimal digits

### IPv6 with Port Numbers

When including TCP/UDP port after IPv6 address, enclose IPv6 address in square brackets:
```
[2001:db8:0:10::1]:80
```

## IPv6 Subnets

### Standard IPv6 Structure

- **Network prefix**: Identifies subnet
- **Interface ID**: Identifies particular interface on subnet
- **Standard subnet mask**: `/64` prefix length
  - 64 bits for network prefix
  - 64 bits for interface ID
  - Single IPv6 subnet can accommodate 2^64 hosts

### IPv6 Address Allocation

- **ISP allocation**: Commonly `/48` prefix to organizations
- **Organizational subnetting**: Remaining 16 bits (64-48=16) for internal subnets
- **Possible subnets**: 2^16 = 65,536 distinct subnets

## Special IPv6 Addresses

| Address/Network | Purpose | Description |
|-----------------|---------|-------------|
| `::1/128` | localhost | IPv6 equivalent to 127.0.0.1/8 |
| `::` | unspecified address | IPv6 equivalent to 0.0.0.0 |
| `::/0` | default route | IPv6 equivalent to 0.0.0.0/0 |
| `2000::/3` | Global unicast | Public, routable IPv6 addresses |
| `fd00::/8` | Unique local (RFC 4193) | Private routable IP address space |
| `fe80::/10` | Link-local | Unroutable addresses for local link |
| `ff00::/8` | Multicast | IPv6 equivalent to 224.0.0.0/4 |

### Link-Local Addresses

- **Unroutable addresses** for same network link only
- **Automatically configured** on every IPv6 interface
- **Always present** regardless of other IPv6 addresses
- **Interface ID generation**: Random but stable (RFC 7217) for privacy

### Multicast in IPv6

- **One-to-many communication**
- **Routable** (unlike broadcast)
- **Key address**: `ff02::1` (all-nodes link-local address)
- **Replaces broadcast** functionality from IPv4

## IPv6 Address Configuration

IPv6 supports three configuration methods:

### 1. Manual Configuration

- Similar to IPv4 static configuration
- **Reserved interface IDs:**
  - `0000:0000:0000:0000` (subnet router anycast)
  - `fdff:ffff:ffff:ff80` through `fdff:ffff:ffff:ffff`

### 2. DHCPv6 (Dynamic Configuration)

- **No broadcast addresses** used
- **Client process:**
  1. Sends requests from link-local address
  2. Uses `ff02::1:2` multicast address (UDP port 547)
  3. DHCPv6 servers respond to UDP port 546
- **Package**: `kea` in RHEL 10

### 3. SLAAC (Stateless Address Autoconfiguration)

- **Process:**
  1. Host configures link-local `fe80::/64` address automatically
  2. Sends router solicitation to `ff02::2` multicast address
  3. IPv6 router responds with network prefix and information
  4. Host combines network prefix with generated interface ID
  5. Router sends regular advertisements to keep settings current
- **Package**: `radvd` (Router Advertisement Daemon)

### Combined Approaches

Some deployments combine SLAAC and DHCPv6:
- **SLAAC**: Provides network address information
- **DHCPv6**: Provides additional options (DNS servers, search domains)

---

# Hostnames and Name Resolution

IP addresses are not human-friendly for daily use. Linux provides name resolution mechanisms to map hostnames to IP addresses.

## Name Resolution Methods

### 1. Static Host Files

- **Location**: `/etc/hosts` file
- **Drawback**: Must manually update on every system
- **Use case**: Small, static environments

### 2. DNS (Domain Name System)

- **Distributed network** of servers
- **Dynamic mapping** of hostnames and IP addresses
- **Configuration**: Host contacts nameserver
- **Settings location**: `/etc/resolv.conf` file
- **Provisioning**: Through DHCP or static configuration

---

# 17.2. Quiz - Networking Concepts

## Questions and Answers

1. **What is the size, in bits, of an IPv4 address?**
   - d. 32 ✓

2. **Which term determines how many leading bits in the IP address contribute to its network address?**
   - b. netmask ✓

3. **Which address represents a valid IPv4 host address on the 192.168.1.0/24 network?**
   - a. 192.168.1.188 ✓

4. **Which number is the size, in bits, of an IPv6 address?**
   - f. 128 ✓

5. **Which address does not represent a valid IPv6 address?**
   - f. 2001:db8::7::2 ✓ (two double colons not allowed)

6. **Which term refers to the ability of one system to send traffic to a special IP address that multiple systems receive?**
   - d. multicast ✓

---

# 17.3. Validating Network Configuration

## Objectives
Test and inspect the current network configuration by using command-line utilities.

## Gathering Network Information

Linux offers robust tools for gathering network information. Red Hat recommends the versatile `ip` command, which provides comprehensive details about network interfaces, statistics, addresses, and routes.

## Identifying Network Devices

### Display Network Interfaces

```bash
user@host:~$ ip link show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN ...
   link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP ...
   link/ether 52:54:00:00:fa:0a brd ff:ff:ff:ff:ff:ff
   altname enp0s3
   altname enx52540000fa0a
3: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 8942 qdisc fq_codel state UP ...
   link/ether 52:54:00:01:fa:1e brd ff:ff:ff:ff:ff:ff
   altname enp0s4
   altname enx52540001fa1e
```

**Key information:**
- **`lo`**: Loopback virtual interface (device communicates with itself)
- **`ens3`, `ens4`**: Ethernet interfaces
- **`link/ether`**: MAC address of device
- **`altname`**: Alternative names for same device

## Displaying IP Addresses

```bash
user@host:~$ ip addr show ens3
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP ...
   link/ether 52:54:00:00:fa:0a brd ff:ff:ff:ff:ff:ff
   altname enp0s3
   altname enx52540000fa0a
   inet 172.25.250.10/24 brd 172.25.250.255 scope global noprefixroute ens3
        valid_lft forever preferred_lft forever
   inet6 2001:db8:0:1:5054:ff:fe00:b/64 scope global
        valid_lft forever preferred_lft forever
   inet6 fe80::5054:ff:fe00:fa0a/64 scope link noprefixroute
        valid_lft forever preferred_lft forever
```

**Key information:**
- **`state UP`**: Active interface
- **`inet`**: IPv4 address with network prefix and scope
- **`inet6` (global scope)**: IPv6 address with unlimited scope (routable)
- **`inet6` (link scope)**: Link-local IPv6 address (automatically generated)

## Displaying Performance Statistics

```bash
user@host:~$ ip -s link show ens3
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP ...
   link/ether 52:54:00:00:fa:0a brd ff:ff:ff:ff:ff:ff
   RX: bytes  packets  errors  dropped  missed   mcast
   10398126    75186       0        3       0       0
   TX: bytes  packets  errors  dropped  carrier collsns
   22214542    68283       0        0       0       0
   altname enp0s3
   altname enx52540000fa0a
```

**Statistics help detect network issues:**
- **RX**: Received packet counts, errors, dropped packets
- **TX**: Transmitted packet counts, errors, dropped packets

## Verifying Connectivity Between Hosts

### IPv4 Connectivity Testing

```bash
user@host:~$ ping -c3 192.0.2.254
PING 192.0.2.1 (192.0.2.254) 56(84) bytes of data.
64 bytes from 192.0.2.254: icmp_seq=1 ttl=64 time=4.33 ms
64 bytes from 192.0.2.254: icmp_seq=2 ttl=64 time=3.48 ms
64 bytes from 192.0.2.254: icmp_seq=3 ttl=64 time=6.83 ms

--- 192.0.2.254 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 3.485/4.885/6.837/1.424 ms
```

### IPv6 Connectivity Testing

```bash
user@host:~$ ping6 2001:db8:0:1::1
PING 2001:db8:0:1::1(2001:db8:0:1::1) 56 data bytes
64 bytes from 2001:db8:0:1::1: icmp_seq=1 ttl=64 time=18.4 ms
64 bytes from 2001:db8:0:1::1: icmp_seq=2 ttl=64 time=0.178 ms
64 bytes from 2001:db8:0:1::1: icmp_seq=3 ttl=64 time=0.180 ms
^C
--- 2001:db8:0:1::1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2001ms
rtt min/avg/max/mdev = 0.178/6.272/18.458/8.616 ms
```

### IPv6 Link-Local Addresses

For link-local addresses, specify network interface with scope identifier:

```bash
user@host:~$ ping6 -c 1 fe80::f482:dbff:fe25:6a9f%ens4
PING fe80::f482:dbff:fe25:6a9f%ens4(fe80::f482:dbff:fe25:6a9f) 56 data bytes
64 bytes from fe80::f482:dbff:fe25:6a9f: icmp_seq=1 ttl=64 time=22.9 ms

--- fe80::f482:dbff:fe25:6a9f%ens4 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 22.903/22.903/22.903/0.000 ms
```

### SSH with IPv6 Link-Local

```bash
user@host:~$ ssh fe80::f482:dbff:fe25:6a9f%ens4
user@fe80::f482:dbff:fe25:6a9f%ens4's password:
```

### IPv6 Network Discovery

Discover other IPv6 hosts on local network:

```bash
user@host:~$ ping6 ff02::1%ens4
PING ff02::1%ens4(ff02::1) 56 data bytes
64 bytes from fe80::78cf:7fff:fed2:f97b: icmp_seq=1 ttl=64 time=22.7 ms
64 bytes from fe80::f482:dbff:fe25:6a9f: icmp_seq=1 ttl=64 time=30.1 ms (DUP!)
```

## Troubleshooting Routing Issues

### Displaying Routing Tables

#### IPv4 Routing Table

```bash
user@host:~$ ip route
default via 192.0.2.254 dev ens3 proto static metric 1024
192.0.2.0/24 dev ens3 proto kernel scope link src 192.0.2.2
10.0.0.0/8 dev ens4 proto kernel scope link src 10.0.0.11
```

**Routing logic:**
- Packets not for `192.0.2.0/24` or `10.0.0.0/8` go to default router `192.0.2.254`
- Packets for `192.0.2.0/24` sent directly via `ens3`
- Packets for `10.0.0.0/8` sent directly via `ens4`

#### IPv6 Routing Table

```bash
user@host:~$ ip -6 route
2001:db8:0:1::/64 dev ens3 proto kernel metric 101 pref medium
fe80::/64 dev ens3 proto kernel metric 1024 pref medium
fe80::/64 dev ens4 proto kernel metric 1024 pref medium
default via 2001:db8:0:1::ffff dev ens3 proto static metric 101 pref medium
```

### Tracing Traffic Routes

#### Using tracepath

```bash
user@host:~$ tracepath access.redhat.com
...output omitted...
4: 71-32-28-145.rcmt.qwest.net                    48.853ms asymm  5
5: dcp-brdr-04.inet.qwest.net                    100.732ms asymm  7
6: 206.111.0.153.ptr.us.xo.net                   96.245ms asymm  7
7: 207.88.14.162.ptr.us.xo.net                   85.270ms asymm  8
8: ae1d0.cir1.atlanta6-ga.us.xo.net              64.160ms asymm  7
9: 216.156.108.98.ptr.us.xo.net                 108.652ms
10: bu-ether13.atlngamq46w-bcr00.tbone.rr.com    107.286ms asymm 12
...output omitted...
```

#### Using tracepath6 for IPv6

```bash
user@host:~$ tracepath6 2001:db8:0:2::451
1?: [LOCALHOST]                                    0.091ms pmtu 1500
1:  2001:db8:0:1::ba                               0.214ms
2:  2001:db8:0:1::1                                0.512ms
3:  2001:db8:0:2::451                              0.559ms reached
    Resume: pmtu 1500 hops 3 back 3
```

#### Using mtr (Interactive/Report Mode)

```bash
user@host:~$ mtr -r -c 5 access.redhat.com
Start: 2025-05-29T22:41:52+0000
HOST: servera                     Loss%   Snt   Last   Avg  Best  Wrst StDev
  1.|-- classroom.lab.example.com  0.0%     5    0.4   0.4   0.3   0.5   0.1
  2.|-- 72.32.49.3                 0.0%     5    1.0   1.3   0.9   2.3   0.6
  3.|-- 72.32.28.27                0.0%     5    1.3   1.5   1.3   1.7   0.2
  4.|-- aggr172b-54-cored.dfw3.ra  0.0%     5    1.0   1.0   0.8   1.0   0.1
  ...output omitted...
```

## Troubleshooting Port and Service Issues

Applications use **sockets** as communication endpoints (IP address + protocol + port number).

### Displaying Socket Statistics

The `ss` command displays socket statistics (supersedes `netstat`):

```bash
user@host:~$ ss -tan
State    Recv-Q Send-Q Local Address:Port    Peer Address:Port
LISTEN   0      100    127.0.0.1:25         0.0.0.0:*
LISTEN   0      128    0.0.0.0:22           0.0.0.0:*
LISTEN   0      4096   0.0.0.0:111          0.0.0.0:*
ESTAB    0      52     172.25.250.10:22     172.25.250.9:57560
LISTEN   0      100    [::1]:25             [::]:*
LISTEN   0      4096   *:9090               *:*
LISTEN   0      128    [::]:22              [::]:*
LISTEN   0      4096   [::]:111             [::]:*
```

**Analysis:**
- **SMTP mail service** (port 25): Only on IPv4 loopback (127.0.0.1) - not network accessible
- **SSH service** (port 22): Listening on all IPv4 addresses (0.0.0.0) and IPv6 ([::])
- **Established SSH connection**: From 172.25.250.9 to this host
- **Web Console** (port 9090): Listening on all interfaces

### Common ss/netstat Options

| Option | Description |
|--------|-------------|
| `-n` | Show numbers instead of names for interfaces and ports |
| `-t` | Show TCP sockets |
| `-u` | Show UDP sockets |
| `-l` | Show only listening sockets |
| `-a` | Show all (listening and established) sockets |
| `-p` | Show the process that uses the sockets |
| `-A inet` | Display active connections for inet address family |

---

# 17.4. Guided Exercise - Validate Network Configuration

## Objectives
- Identify current network interfaces and network addresses
- Test network connectivity and routing

## Lab Instructions

### Prerequisites
```bash
student@workstation:~$ lab start netbasics-validate
```

### Tasks

1. **Display network interfaces**
   ```bash
   [student@servera ~]$ ip link
   1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN ...
      link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
   2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP ...
      link/ether 52:54:00:00:fa:0a brd ff:ff:ff:ff:ff:ff
      altname enp0s3
      altname enx52540000fa0a
   3: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 8942 qdisc fq_codel state UP ...
      link/ether 52:54:00:01:fa:1e brd ff:ff:ff:ff:ff:ff
      altname enp0s4
      altname enx52540001fa1e
   ```

2. **Display IP addresses for all interfaces**
   ```bash
   [student@servera ~]$ ip -br addr
   lo               UNKNOWN        127.0.0.1/8 ::1/128
   ens3             UP             172.25.250.10/24 fe80::5054:ff:fe00:fa0a/64
   ens4             UP
   ```

3. **Display packet statistics**
   ```bash
   [student@servera ~]$ ip -s link show ens3
   2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP
   ...
   link/ether 52:54:00:00:fa:0a brd ff:ff:ff:ff:ff:ff
       RX: bytes  packets  errors  dropped  missed   mcast
        1214412     7146       0        3       0       0
       TX: bytes  packets  errors  dropped  carrier collsns
        2099808     6288       0        0       0       0
   ```

4. **View routing table**
   ```bash
   [student@servera ~]$ ip route
   default via 172.25.250.254 dev ens3 proto static metric 100
   172.25.250.0/24 dev ens3 proto kernel scope link src 172.25.250.10 metric 100
   ```

5. **Verify router accessibility**
   ```bash
   [student@servera ~]$ ping -c3 172.25.250.254
   PING 172.25.250.254 (172.25.250.254) 56(84) bytes of data.
   64 bytes from 172.25.250.254: icmp_seq=1 ttl=64 time=0.196 ms
   64 bytes from 172.25.250.254: icmp_seq=2 ttl=64 time=0.436 ms
   64 bytes from 172.25.250.254: icmp_seq=3 ttl=64 time=0.361 ms
   
   --- 172.25.250.254 ping statistics ---
   3 packets transmitted, 3 received, 0% packet loss, time 49ms
   rtt min/avg/max/mdev = 0.196/0.331/0.436/0.100 ms
   ```

6. **Trace route to destination**
   ```bash
   [student@servera ~]$ tracepath classroom.example.com
   1?: [LOCALHOST]                      pmtu 1500
   1:  classroom.example.com           0.296ms reached
   1:  classroom.example.com           0.125ms reached
       Resume: pmtu 1500 hops 1 back 1
   ```

7. **Display listening TCP sockets**
   ```bash
   [student@servera ~]$ ss -lt
   State      Recv-Q Send-Q Local Address:Port    Peer Address:Port
   LISTEN     0      128    0.0.0.0:ssh          0.0.0.0:*
   LISTEN     0      4096   0.0.0.0:sunrpc       0.0.0.0:*
   LISTEN     0      128    [::]:ssh             [::]:*
   LISTEN     0      4096   [::]:sunrpc          [::]:*
   LISTEN     0      4096   *:websm              *:*
   ```

8. **Clean up**
   ```bash
   student@workstation:~$ lab finish netbasics-validate
   ```

---

# 17.5. Summary

## Key Learning Points

### TCP/IP Fundamentals
- **Four-layer model**: Application, Transport, Internet, Link
- **Protocols**: TCP (reliable), UDP (fast), IPv4, IPv6
- **Addressing**: IP addresses, ports, sockets

### Network Interface Naming
- **Consistent naming**: Based on firmware and PCI topology
- **Predictable**: Persists across reboots
- **Format**: Type + location (e.g., `ens3`, `enp0s1f0`)

### IPv4 Networking
- **32-bit addresses**: Four dot-separated octets
- **Subnetting**: Network prefix + host identifier
- **Configuration**: DHCP (dynamic) or static
- **Tools**: `ipcalc` for calculations

### IPv6 Networking
- **128-bit addresses**: Eight colon-separated hexadecimal groups
- **Standard prefix**: `/64` for most networks
- **Configuration**: Manual, DHCPv6, or SLAAC
- **Special addresses**: Link-local, multicast, global unicast

### Network Troubleshooting Tools

| Tool | Purpose | Usage |
|------|---------|-------|
| `ip` | Display/configure interfaces, addresses, routes | `ip addr show`, `ip route` |
| `ping`/`ping6` | Test connectivity | `ping -c3 192.0.2.1` |
| `tracepath`/`traceroute` | Trace route to destination | `tracepath example.com` |
| `mtr` | Interactive route tracing | `mtr -r -c5 example.com` |
| `ss` | Display socket statistics | `ss -tan`, `ss -lt` |

### Best Practices
- Use consistent interface naming for predictability
- Understand routing table logic for troubleshooting
- Test connectivity at multiple layers (link, network, transport)
- Monitor network statistics for performance issues
- Document network configurations for maintenance

## References
- **Man pages**: ip(8), ping(8), tracepath(8), traceroute(8), mtr(8), ss(8)
- **RFCs**: 1122 (TCP/IP), 1918 (Private addresses), 4193 (IPv6 ULA), 7217 (IPv6 SLAAC)
- **Red Hat Documentation**: Configuring and Managing Networking guide
