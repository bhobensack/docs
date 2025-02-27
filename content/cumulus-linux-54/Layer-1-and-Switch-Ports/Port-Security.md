---
title: Port Security
author: NVIDIA
weight: 380
toc: 3
---
Port security is a layer 2 traffic control feature that enables you to manage network access from end-users. Use port security to:

- Limit port access to specific MAC addresses so that the port does not forward ingress traffic from source addresses that are not defined.  
- Limit port access to only the first learned MAC address on the port (sticky MAC) so that the device with that MAC address has full bandwidth. You can provide a timeout so that the MAC address on that port no longer has access after a specified time.
- Limit port access to a specific number of MAC addresses.

You can specify what action to take when there is a port security violation (drop packets or put the port into ADMIN down state) and add a timeout for the action to take effect.

{{%notice note%}}
- Layer 2 interfaces in trunk or access mode are currently supported. However, interfaces in a bond are *not* supported.
- NVUE commands are not available for port security configuration.
{{%/notice%}}

## Configure Port Security

To configure port security, add the configuration settings you want to use to the `/etc/cumulus/switchd.d/port_security.conf` file, then restart `switchd` to apply the changes.

| <div style="width:460px">Setting | Description|
| --------| -----------|
| `interface.<port>.port_security.enable` | 1 enables security on the port. 0 disables security on the port.|
| `interface.<port>.port_security.mac_limit` | The maximum number of MAC addresses allowed to access the port. You can specify a number between 0 and 512. The default is 32.|
| `interface.<port>.port_security.static_mac` | The specific MAC addresses allowed to access the port. You can specify multiple MAC addresses. Separate each MAC address with a space.|
| `interface.<port>.port_security.sticky_mac` | 1 enables sticky MAC, where the first learned MAC address on the port is the only MAC address allowed. 0 disables sticky MAC. |
| `interface.<port>.port_security.sticky_timeout` |The time period after which the first learned MAC address ages out and no longer has access to the port. The default aging timeout value is 30 minutes. You can specify a value between 0 and 60 minutes.|
| `interface.<port>.port_security.sticky_aging` | 1 enables sticky MAC aging. 0 disables sticky MAC aging.|
| `interface.<port>.port_security.violation_mode` | The violation mode: 0 (shutdown) puts a port into ADMIN down state. 1 (restrict) drops packets.|
| `interface.<port>.port_security.violation_timeout` | The number of seconds after which the violation mode times out. You can specify a value between 0 and 3600 seconds. The default value is 1800 seconds.|

The following example shows an `/etc/cumulus/switchd.d/port_security.conf` configuration file:

```
cumulus@switch:~$ sudo nano /etc/cumulus/switchd.d/port_security.conf
interface.swp1.port_security.enable = 1
interface.swp1.port_security.mac_limit = 32
interface.swp1.port_security.static_mac = 00:02:00:00:00:05 00:02:00:00:00:06
interface.swp1.port_security.sticky_mac = 1
interface.swp1.port_security.sticky_timeout = 2000
interface.swp1.port_security.sticky_aging = 1
interface.swp1.port_security.violation_mode = 0
interface.swp1.port_security.violation_timeout = 3600
...
```
