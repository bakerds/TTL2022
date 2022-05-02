# Tech Talk Live 2022

# Basic Networking Design and Configuration Lab


## Switch configuration

### Getting connected to the switch
<details>
<summary>Windows</summary>

- Connect your serial console cable to the switch Console port
- Determine the name of your serial adapter
    - Open Device Manager
    - Expand the **Ports (COM & LPT)** section
    - Take note of the serial port listed (**COM3**, for example)
- Launch your favorite terminal emulator (**[PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)**, for example)
- Choose **Serial**, enter your COM port name, and connect at **9600** baud
</details>
<details>
<summary>macOS / Linux</summary>

- Connect your serial console cable to the switch Console port
- Determine the name of your serial adapter
    - Open a terminal
    - Type `ls /dev/tty.*`
    - Look for a USB tty and note its name (`/dev/ttyUSB0`, for example)
- Launch your favorite terminal emulator (**minicom** or **screen**, for example) and connect to the desired TTY at 9600 baud
    - `screen /dev/ttyUSB0 9600`
</details>

## Logging in to the switch CLI
- Hit enter to start a serial session
- Log in to the switch:
```
    login: admin
    Password: TTL2022

    Last login: Thu Jun 24 21:09:10

    --- JUNOS 20.4R3.8 Kernel 32-bit  JNPR-11.0-20210618.f43645e_buil
    {master:0}
    admin@EX2300C-TTL-13>
```

## The JunOS CLI

Basic commands to run in operational mode (`>` prompt)

- `show configuration` -- show entire running configuration
- `show interfaces terse` -- show interface status
- `configure` -- enter configuration mode

Basic commands to use in configuration mode (`#` prompt)
- `show` -- show the configuration at the current hierarchy
- `set` -- create an element in the configuration
- `delete` -- delete an element from the configuration
- `commit` -- commit changes from candidate config to running config*

\* JunOS uses a two-stage configuration. When editing the configuration, changes are not applied until they are committed. This enables you to prepare config changes without disrupting the current function of the device, and then apply them all at once.

> **Tip:** Typing `?` anywhere in the JunOS CLI will enumerate possible commands or completions from your current position.

## The lab network

### Physical connections
Each "building" (lab group) switch connects to the core via its `ge-0/0/0` interface. The corresponding core switch ports are listed below.


### Transit VLANs and routing
Each building has a separate transit VLAN with an IPv4 and IPv6 subnet used for routing. The building will communicate with the core using traffic tagged with its transit VLAN. In the transit subnets, the core is assigned the first host address, and the building switch is assigned the second host address.

For the subnet `10.95.0.0/30`, the host addresses are `10.95.0.1` (core) and `10.95.0.2` (building).

For the subnet `2620:1d5:c04::/127`, the host addresses are `2620:1d5:c04::` (core) and `2620:1d5:c04::1` (building).


### Address allocations
Each building has an assigned IPv4 and IPv6 prefix. Each prefix is routed to the corresponding building in the core. Client subnets will be chosen from each building's allocation.


### Summary Table

|Group/<br>Building |Core Switch Port |Transit VLAN |IPv4 Transit |IPv6 Transit |IPv4 Allocation |IPv6 Allocation |
|--|--|--|--|--|--|--|
|1  |`ge-0/0/0`  |3001 |`10.95.0.4/30`  |`2620:1d5:c04:1::/127` |`10.95.16.0/20`  |`2620:1d5:c04:1000::/52` |
|2  |`ge-0/0/1`  |3002 |`10.95.0.8/30`  |`2620:1d5:c04:2::/127` |`10.95.32.0/20`  |`2620:1d5:c04:2000::/52` |
|3  |`ge-0/0/2`  |3003 |`10.95.0.12/30` |`2620:1d5:c04:3::/127` |`10.95.48.0/20`  |`2620:1d5:c04:3000::/52` |
|4  |`ge-0/0/3`  |3004 |`10.95.0.16/30` |`2620:1d5:c04:4::/127` |`10.95.64.0/20`  |`2620:1d5:c04:4000::/52` |
|5  |`ge-0/0/4`  |3005 |`10.95.0.20/30` |`2620:1d5:c04:5::/127` |`10.95.80.0/20`  |`2620:1d5:c04:5000::/52` |
|6  |`ge-0/0/5`  |3006 |`10.95.0.24/30` |`2620:1d5:c04:6::/127` |`10.95.96.0/20`  |`2620:1d5:c04:6000::/52` |
|7  |`ge-0/0/6`  |3007 |`10.95.0.28/30` |`2620:1d5:c04:7::/127` |`10.95.112.0/20` |`2620:1d5:c04:7000::/52` |
|8  |`ge-0/0/7`  |3008 |`10.95.0.32/30` |`2620:1d5:c04:8::/127` |`10.95.128.0/20` |`2620:1d5:c04:8000::/52` |
|9  |`ge-0/0/8`  |3009 |`10.95.0.36/30` |`2620:1d5:c04:9::/127` |`10.95.144.0/20` |`2620:1d5:c04:9000::/52` |
|10 |`ge-0/0/9`  |3010 |`10.95.0.40/30` |`2620:1d5:c04:a::/127` |`10.95.160.0/20` |`2620:1d5:c04:a000::/52` |
|11 |`ge-0/0/10` |3011 |`10.95.0.44/30` |`2620:1d5:c04:b::/127` |`10.95.176.0/20` |`2620:1d5:c04:b000::/52` |
|12 |`ge-0/0/11` |3012 |`10.95.0.48/30` |`2620:1d5:c04:c::/127` |`10.95.192.0/20` |`2620:1d5:c04:c000::/52` |

<details>
<summary>Hint: Core static route config</summary>

```
routing-options {
    rib inet6.0 {
        static {
            route ::/0 next-hop 2620:1d5:fff:9510::;
            route 2620:1d5:c04:1000::/52 next-hop 2620:1d5:c04:1::1;
            route 2620:1d5:c04:2000::/52 next-hop 2620:1d5:c04:2::1;
            route 2620:1d5:c04:3000::/52 next-hop 2620:1d5:c04:3::1;
            route 2620:1d5:c04:4000::/52 next-hop 2620:1d5:c04:4::1;
            route 2620:1d5:c04:5000::/52 next-hop 2620:1d5:c04:5::1;
            route 2620:1d5:c04:6000::/52 next-hop 2620:1d5:c04:6::1;
            route 2620:1d5:c04:7000::/52 next-hop 2620:1d5:c04:7::1;
            route 2620:1d5:c04:8000::/52 next-hop 2620:1d5:c04:8::1;
            route 2620:1d5:c04:9000::/52 next-hop 2620:1d5:c04:9::1;
            route 2620:1d5:c04:a000::/52 next-hop 2620:1d5:c04:a::1;
            route 2620:1d5:c04:b000::/52 next-hop 2620:1d5:c04:b::1;
            route 2620:1d5:c04:c000::/52 next-hop 2620:1d5:c04:c::1;
            route 2620:1d5:c04:d000::/52 next-hop 2620:1d5:c04:d::1;
        }
    }
    static {
        route 0.0.0.0/0 next-hop 10.16.95.1;
        route 10.95.16.0/20 next-hop 10.95.0.2;
        route 10.95.32.0/20 next-hop 10.95.0.6;
        route 10.95.48.0/20 next-hop 10.95.0.10;
        route 10.95.64.0/20 next-hop 10.95.0.14;
        route 10.95.80.0/20 next-hop 10.95.0.18;
        route 10.95.96.0/20 next-hop 10.95.0.22;
        route 10.95.112.0/20 next-hop 10.95.0.26;
        route 10.95.128.0/20 next-hop 10.95.0.30;
        route 10.95.144.0/20 next-hop 10.95.0.34;
        route 10.95.160.0/20 next-hop 10.95.0.38;
        route 10.95.176.0/20 next-hop 10.95.0.42;
        route 10.95.192.0/20 next-hop 10.95.0.46;
        route 10.95.208.0/20 next-hop 10.95.0.50;
    }
}
```

</details>

## Diagrams

<details>
<summary>Overview</summary>

![Overview](img/TTL%20Lab%20Network%20-%20Basic%20Diagram.png?raw=true "Overview")
</details>

<details>
<summary>IPv4 Transits</summary>

![IPv4 Transits](img/TTL%20Lab%20Network%20-%20Transits%20-%20IPv4.png?raw=true "IPv4 Transits")
</details>

<details>
<summary>IPv6 Transits</summary>

![IPv6 Transits](img/TTL%20Lab%20Network%20-%20Transits%20-%20IPv6.png?raw=true "IPv6 Transits")
</details>

<details>
<summary>Address Allocations</summary>

![Address Allocations](img/TTL%20Lab%20Network%20-%20Address%20Space%20Assignments.png?raw=true "Address Allocations")
</details>

## Sample Configs
Below you can find the relevant excerpts from Building 13's switch config, and adapt them to your building.

### Building 13 Transits

Note that the config is displayed in a hierarchical fashion, but it is created using set commands. Both forms are shown here for your convenience, along with annotations explaining the purpose of each statement.

After making changes to the candidate config, don't forget to issue `commit` to apply your changes!


#### Create VLANs

```
vlans {
    v3013_Group_13_Transit {   ## Name of the VLAN
        vlan-id 3013;          ## VLAN ID
        l3-interface irb.3013; ## Create a layer 3 routing interface for this VLAN
    }
}
```

```
set vlans v3013_Group_13_Transit vlan-id 3013
set vlans v3013_Group_13_Transit l3-interface irb.3013
```


#### Configure uplink interface with VLAN 3013 tagged membership

```
interfaces {
    ge-0/0/0 {
        unit 0 {                        ## For layer 2 ports, unit 0 is usually the only subinterface used
            family ethernet-switching {
                interface-mode trunk;   ## Configure port to accept/transmit tagged frames
                vlan {
                    members 3013;       ## Limit this port to frames tagged with VLAN 3013
                }
            }
        }
    }
}
```

```
set interfaces ge-0/0/0 unit 0 family ethernet-switching interface-mode trunk
set interfaces ge-0/0/0 unit 0 family ethernet-switching vlan members 3013
```

The term "unit" may be replaced by a period, as in:

```
set interfaces ge-0/0/0.0 family ethernet-switching interface-mode trunk
set interfaces ge-0/0/0.0 family ethernet-switching vlan members 3013
```


#### Configure layer 3 routing interface for transit VLAN 3013

```
interfaces {
    irb {
        unit 3013 {                            ## This is the "irb.3013" mentioned in the VLAN config
            family inet {
                address 10.95.0.54/30;         ## IPv4 transit address`
            }
            family inet6 {
                address 2620:1d5:c04:d::1/127; ## IPv6 transit subnet
            }
        }
    }
}
```

```
set interfaces irb unit 3013 family inet address 10.95.0.54/30
set interfaces irb unit 3013 family inet6 address 2620:1d5:c04:d::1/127
```

These set commands can also be written using the shortened name of the layer 3 interfaces:

```
set interfaces irb.3013 family inet address 10.95.0.54/30
set interfaces irb.3013 family inet6 address 2620:1d5:c04:d::1/127
```


#### Create default routes towards the core
```
routing-options {
    rib inet6.0 {
        static {
            route ::/0 next-hop 2620:1d5:c04:d::;  ## ::/0 represents all IPv6 destinations
        }
    }
    static {
        route 0.0.0.0/0 next-hop 10.95.0.53;       ## 0.0.0.0/0 represents all IPv4 destinations
    }
}
```

```
set routing-options rib inet6.0 static route ::/0 next-hop 2620:1d5:c04:d::
set routing-options static route 0.0.0.0/0 next-hop 10.95.0.53
```


### Building 13 client subnets

<details>
<summary>Building 13 IPAM</summary>

|VLAN |Name |IPv4 |IPv6 |
|--|--|--|--|
|101 |Network Management     |`10.95.208.0/24` |`2620:1d5:c04:d101::/64` |
|102 |Wireless Management    |`10.95.209.0/24` |`2620:1d5:c04:d102::/64` |
|103 |Hypervisor Management  |`10.95.210.0/24` |`2620:1d5:c04:d103::/64` |
|104 |Servers                |`10.95.211.0/24` |`2620:1d5:c04:d104::/64` |
|105 |DMZ                    |`10.95.212.0/24` |`2620:1d5:c04:d105::/64` |
|106 |Security Devices       |`10.95.213.0/24` |`2620:1d5:c04:d106::/64` |
|107 |Security Cameras       |`10.95.214.0/24` |`2620:1d5:c04:d107::/64` |
|108 |HVAC Controls          |`10.95.215.0/24` |`2620:1d5:c04:d108::/64` |
|109 |Phones                 |`10.95.216.0/24` |`2620:1d5:c04:d109::/64` |
|110 |IT Staff               |`10.95.217.0/24` |`2620:1d5:c04:d110::/64` |
|111 |Administrative Staff   |`10.95.218.0/24` |`2620:1d5:c04:d111::/64` |
|112 |Staff                  |`10.95.219.0/24` |`2620:1d5:c04:d112::/64` |
|113 |Student Wired          |`10.95.220.0/24` |`2620:1d5:c04:d113::/64` |
|114 |Student Wireless       |`10.95.222.0/23` |`2620:1d5:c04:d114::/64` |

</details>

#### Create VLANs

```
vlans {
    v0110_IT_Staff {
        vlan-id 110;
        l3-interface irb.110;
    }
    v0111_Admin_Staff {
        vlan-id 111;
        l3-interface irb.111;
    }
    v0112_Staff {
        vlan-id 112;
        l3-interface irb.112;
    }
    v0113_Student_Wired {
        vlan-id 113;
        l3-interface irb.113;
    }
    v0114_Student_Wireless {
        vlan-id 114;
        l3-interface irb.114;
    }
}
```

```
set vlans v0110_IT_Staff vlan-id 110
set vlans v0110_IT_Staff l3-interface irb.110
set vlans v0111_Admin_Staff vlan-id 111
set vlans v0111_Admin_Staff l3-interface irb.111
set vlans v0112_Staff vlan-id 112
set vlans v0112_Staff l3-interface irb.112
set vlans v0113_Student_Wired vlan-id 113
set vlans v0113_Student_Wired l3-interface irb.113
set vlans v0114_Student_Wireless vlan-id 114
set vlans v0114_Student_Wireless l3-interface irb.114
```


#### Configure layer 3 interfaces

```
interfaces {
    irb {
        unit 110 {
            family inet {
                address 10.95.217.1/24;
            }
            family inet6 {
                address 2620:1d5:c04:d110::1/64 {
                    subnet-router-anycast;
                }
            }
        }
        unit 111 {
            family inet {
                address 10.95.218.1/24;
            }
            family inet6 {
                address 2620:1d5:c04:d111::1/64 {
                    subnet-router-anycast;
                }
            }
        }
        unit 112 {
            family inet {
                address 10.95.219.1/24;
            }
            family inet6 {
                address 2620:1d5:c04:d112::1/64 {
                    subnet-router-anycast;
                }
            }
        }
        unit 113 {
            family inet {
                address 10.95.220.1/24;
            }
            family inet6 {
                address 2620:1d5:c04:d113::1/64 {
                    subnet-router-anycast;
                }
            }
        }
        unit 114 {
            family inet {
                address 10.95.222.1/23;
            }
            family inet6 {
                address 2620:1d5:c04:d114::1/64 {
                    subnet-router-anycast;
                }
            }
        }
    }
}
```

```
set interfaces irb unit 110 family inet address 10.95.217.1/24
set interfaces irb unit 110 family inet6 address 2620:1d5:c04:d110::1/64 subnet-router-anycast
set interfaces irb unit 111 family inet address 10.95.218.1/24
set interfaces irb unit 111 family inet6 address 2620:1d5:c04:d111::1/64 subnet-router-anycast
set interfaces irb unit 112 family inet address 10.95.219.1/24
set interfaces irb unit 112 family inet6 address 2620:1d5:c04:d112::1/64 subnet-router-anycast
set interfaces irb unit 113 family inet address 10.95.220.1/24
set interfaces irb unit 113 family inet6 address 2620:1d5:c04:d113::1/64 subnet-router-anycast
set interfaces irb unit 114 family inet address 10.95.222.1/23
set interfaces irb unit 114 family inet6 address 2620:1d5:c04:d114::1/64 subnet-router-anycast
```


#### Configure DHCP relay for client configuration

```
forwarding-options {
    dhcp-relay {
        server-group {
            Central_DHCP {
                10.95.1.2;
            }
        }
        group Central_DHCP {
            active-server-group Central_DHCP allow-server-change;
            interface irb.110;
            interface irb.111;
            interface irb.112;
            interface irb.113;
            interface irb.114;
        }
    }
}
```

```
set forwarding-options dhcp-relay server-group Central_DHCP 10.95.1.2
set forwarding-options dhcp-relay group Central_DHCP active-server-group Central_DHCP
set forwarding-options dhcp-relay group Central_DHCP active-server-group allow-server-change
set forwarding-options dhcp-relay group Central_DHCP interface irb.110
set forwarding-options dhcp-relay group Central_DHCP interface irb.111
set forwarding-options dhcp-relay group Central_DHCP interface irb.112
set forwarding-options dhcp-relay group Central_DHCP interface irb.113
set forwarding-options dhcp-relay group Central_DHCP interface irb.114
```


#### Configure router advertisements for IPv6 client auto-configuration

```
protocols {
    router-advertisement {
        interface irb.110 {
            prefix 2620:1d5:c04:d110::/64;
        }
        interface irb.111 {
            prefix 2620:1d5:c04:d111::/64;
        }
        interface irb.112 {
            prefix 2620:1d5:c04:d112::/64;
        }
        interface irb.113 {
            prefix 2620:1d5:c04:d113::/64;
        }
        interface irb.114 {
            prefix 2620:1d5:c04:d114::/64;
        }
    }
}
```

```
set protocols router-advertisement interface irb.110 prefix 2620:1d5:c04:d110::/64
set protocols router-advertisement interface irb.111 prefix 2620:1d5:c04:d111::/64
set protocols router-advertisement interface irb.112 prefix 2620:1d5:c04:d112::/64
set protocols router-advertisement interface irb.113 prefix 2620:1d5:c04:d113::/64
set protocols router-advertisement interface irb.114 prefix 2620:1d5:c04:d114::/64
```


#### Configure access ports for user devices

```
interfaces {
    ge-0/0/2 {
        description "IT staff computer";
        unit 0 {
            family ethernet-switching {
                interface-mode access;
                vlan {
                    members 110;
                }
            }
        }
    }
    ge-0/0/3 {
        description "Principal PC";
        unit 0 {
            family ethernet-switching {
                interface-mode access;
                vlan {
                    members 111;
                }
            }
        }
    }
    ge-0/0/4 {
        description "Room 104 Teacher Dock";
        unit 0 {
            family ethernet-switching {
                interface-mode access;
                vlan {
                    members 112;
                }
            }
        }
    }
    ge-0/0/5 {
        description "CAD Lab 220 Workstation 1";
        unit 0 {
            family ethernet-switching {
                interface-mode access;
                vlan {
                    members 113;
                }
            }
        }
    }
}
```

```
set interfaces ge-0/0/2 description "IT staff computer"
set interfaces ge-0/0/2 unit 0 family ethernet-switching interface-mode access
set interfaces ge-0/0/2 unit 0 family ethernet-switching vlan members 110
set interfaces ge-0/0/3 description "Principal PC"
set interfaces ge-0/0/3 unit 0 family ethernet-switching interface-mode access
set interfaces ge-0/0/3 unit 0 family ethernet-switching vlan members 111
set interfaces ge-0/0/4 description "Room 104 Teacher Dock"
set interfaces ge-0/0/4 unit 0 family ethernet-switching interface-mode access
set interfaces ge-0/0/4 unit 0 family ethernet-switching vlan members 112
set interfaces ge-0/0/5 description "CAD Lab 220 Workstation 1"
set interfaces ge-0/0/5 unit 0 family ethernet-switching interface-mode access
set interfaces ge-0/0/5 unit 0 family ethernet-switching vlan members 113
```

### Check in with Dan to create IPv4 DHCP scopes on the lab DHCP server

---

### Testing / Troubleshooting

#### After creating the building transits, see if you can ping the other end of the transit:

(First `exit` config mode)

```
admin@EX2300C-TTL-13> ping 10.95.0.53
PING 10.95.0.53 (10.95.0.53): 56 data bytes
64 bytes from 10.95.0.53: icmp_seq=0 ttl=64 time=20.270 ms
64 bytes from 10.95.0.53: icmp_seq=1 ttl=64 time=22.505 ms
64 bytes from 10.95.0.53: icmp_seq=2 ttl=64 time=9.015 ms
64 bytes from 10.95.0.53: icmp_seq=3 ttl=64 time=10.573 ms
64 bytes from 10.95.0.53: icmp_seq=4 ttl=64 time=18.764 ms
^C
--- 10.95.0.53 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max/stddev = 9.015/16.225/22.505/5.407 ms

{master:0}
admin@EX2300C-TTL-13>
```

##### Don't forget to test both protocols:
```
admin@EX2300C-TTL-13> ping 2620:1d5:c04:d::
PING6(56=40+8+8 bytes) 2620:1d5:c04:d::1 --> 2620:1d5:c04:d::
16 bytes from 2620:1d5:c04:d::, icmp_seq=0 hlim=64 time=7.063 ms
16 bytes from 2620:1d5:c04:d::, icmp_seq=1 hlim=64 time=13.771 ms
16 bytes from 2620:1d5:c04:d::, icmp_seq=2 hlim=64 time=11.726 ms
16 bytes from 2620:1d5:c04:d::, icmp_seq=3 hlim=64 time=7.110 ms
16 bytes from 2620:1d5:c04:d::, icmp_seq=4 hlim=64 time=9.727 ms
^C
--- 2620:1d5:c04:d:: ping6 statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max/std-dev = 7.063/9.879/13.771/2.615 ms

{master:0}
admin@EX2300C-TTL-13>
```


#### If you can ping the other end of the transit, try to ping the world:

Easy-to-remember IPv4 address: Google Public DNS
```
admin@EX2300C-TTL-13> ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8): 56 data bytes
64 bytes from 8.8.8.8: icmp_seq=0 ttl=111 time=32.534 ms
64 bytes from 8.8.8.8: icmp_seq=1 ttl=111 time=30.262 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=111 time=32.002 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=111 time=35.848 ms
^C
--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max/stddev = 30.262/32.662/35.848/2.023 ms

{master:0}
admin@EX2300C-TTL-13>
```

Easy-to-remember IPv6 address: Sprint/T-Mobile website
```
admin@EX2300C-TTL-13> ping 2600::
PING6(56=40+8+8 bytes) 2620:1d5:c04:d::1 --> 2600::
16 bytes from 2600::, icmp_seq=0 hlim=53 time=33.148 ms
16 bytes from 2600::, icmp_seq=1 hlim=53 time=33.870 ms
16 bytes from 2600::, icmp_seq=2 hlim=53 time=34.075 ms
16 bytes from 2600::, icmp_seq=3 hlim=53 time=33.817 ms
16 bytes from 2600::, icmp_seq=4 hlim=53 time=33.710 ms
^C
--- 2600:: ping6 statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max/std-dev = 33.148/33.724/34.075/0.311 ms

{master:0}
admin@EX2300C-TTL-13>
```


#### Connect your PC to a client port and see if you can reach the Internet!

- Be sure to disable your Wi-Fi first.
- Run `ipconfig` or `ip [-6] a` in your terminal to see if you got an IPv4 and IPv6 address in the expected ranges.
- Try to ping some hosts from your PC

Sites to visit:
- [test-ipv6.com](http://www.test-ipv6.com)
- [ipv6.google.com](https://ipv6.google.com)
- [Google IPv6 Statistics](https://www.google.com/intl/en/ipv6/statistics.html)

Check out the browser extension *IPvFoo* to get visual feedback on each website about which resources are loaded over IPv4 or IPv6
- [Chrome Extension](https://chrome.google.com/webstore/detail/ipvfoo/ecanpcehffngcegjmadlcijfolapggal?hl=en)
- [Firefox Extension](https://addons.mozilla.org/en-US/firefox/addon/ipvfoo-pmarks/)


#### Team up with another group and see if you can ping from your PC to theirs and vice-versa

Run route traces from your PC to theirs and see if you can make sense of the results.