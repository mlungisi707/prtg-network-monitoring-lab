# Lab 3: Centralized Network Monitoring Infrastructure with PRTG

## Project Overview

This lab implements a centralized network monitoring solution using **PRTG Network Monitor** to track a virtual two-device topology: a VyOS router and a VyOS switch. The goal was to get real-time visibility into device health, resource usage and interface traffic, and then prove the setup works by breaking something on purpose and confirming that PRTG detects it, alerts on it, and shows recovery.

## Topology

| Role | Device | Address | Monitored interface |
|---|---|---|---|
| Monitoring host | Windows PC running PRTG | 192.168.228.1 | n/a |
| Router | Vyos-Lab-Router (VyOS) | 192.168.228.128 | eth0 |
| Switch | Vyos-Lab-Switch (VyOS) | 192.168.228.131 | eth0 |

* **Hypervisor:** VMware Workstation, NAT mode (VMnet8, 192.168.228.0/24)
* **Monitoring method:** SNMPv2c polling plus ICMP ping, from the PRTG Local Probe

## What Is Monitored

**Per device (router and switch):**

* **Availability:** Ping sensors (response time)
* **Uptime:** SNMP System Uptime
* **CPU:** SNMP CPU Load and CPU Usage
* **Memory:** SNMP Memory / Linux Meminfo
* **Disk:** Disk Free (switch)
* **Traffic:** SNMP Traffic on eth0

**PRTG itself:** Core Health, Probe Health and System Health on the Local Probe.

## Dashboard

A custom live map (**Lab-Network-Dashboard**, built in PRTG Map Designer) shows both devices and their sensors with real-time status colours and an overall sensor status wheel.

## Alerting

Notification triggers are set at the Group Root level and on the router device:

| Condition | Action |
|---|---|
| Sensor Down for 60 seconds | Ticket notification (Root) / Email and push notification to admin (router device) |
| Sensor Down for 60 seconds | Email and push notification to admin (no repeat) |
| Sensor no longer Down | Recovery notification |

## Failure Test

1. **Traffic generation:** Ran `iperf3` between the two VyOS devices (about 4.6 GB at roughly 790 Mbit/s) and confirmed the PRTG traffic sensors showed it.
2. **Controlled failure:** On the router, ran `set interfaces ethernet eth0 disable`, then `commit` and `save`.
3. **Detection and alerting:** PRTG turned the router red on the dashboard, fired a browser push notification ("New Alarms: 6"), and logged ticket #161: *Vyos-Lab-Router (002) eth0 Traffic (SNMP Traffic) Down*.
4. **Recovery:** Ran `delete interfaces ethernet eth0 disable`, then `commit` and `save`. The interface came back up and the dashboard returned to green.

## Evidence

### 1. Router interface (baseline)
![VyOS router eth0 up](screenshots/01-vyos-lab-router-eth0-interface.png)

### 2. Switch interface (baseline)
![VyOS switch eth0 up](screenshots/02-vyos-lab-switch-eth0-interface.png)

### 3. PRTG Local Probe with both devices in the 1st group
![PRTG device tree](screenshots/03-prtg-local-probe-1st-group-devices.png)

### 4. Lab dashboard map (healthy)
![PRTG map, healthy](screenshots/04-prtg-map-lab-dashboard.png)

### 5. Group Root notification triggers
![PRTG notification triggers](screenshots/05-prtg-group-root-notification-triggers.png)

### 6. Disabling eth0 on the router (CLI)
![eth0 disabled on router](screenshots/06-vyos-lab-router-eth0-disabled-cli.png)

### 7. Map showing the router down
![PRTG map, router down](screenshots/07-prtg-map-router-disabled.png)

### 8. Ticket raised for router eth0 down
![PRTG ticket](screenshots/08-prtg-ticket-router-eth0-down.png)

### 9. Bringing eth0 back up (CLI)
![eth0 restored on router](screenshots/09-vyos-lab-router-eth0-restored-cli.png)

### 10. Map showing the router fully restored
![PRTG map, restored](screenshots/10-prtg-map-device-fully-restored.png)

## Key Takeaways

* Built a working SNMP and ping monitoring setup in PRTG for virtual network devices.
* Validated alerting with a real, controlled failure instead of assuming it works.
* Confirmed the full incident cycle: detect, alert, ticket, recover.
