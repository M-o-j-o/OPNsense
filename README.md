# 🔥 OPNsense Lab Docs

A collection of practical, hands-on guides for deploying and configuring **OPNsense** in a virtualized environment using **QEMU/KVM + Virt-Manager**. Built as a technical documentation exercise written clearly enough for engineers at any level to follow.

> ⚠️ These guides assume a virtualized setup but the configurations are applicable to bare-metal deployments as well.

* * *

## 📋 Guides

### 1\. [Installation and Initial Setup](OPNSENSE%20CONFIGURATION/Installation%20and%20initial%20setup/)

Get OPNsense up and running inside a KVM virtual machine. Covers ISO setup, VM configuration, interface assignment, and basic initial configuration via the web GUI.

### 2\. [HTTP Transparent Proxy & Web Filtering](OPNSENSE%20CONFIGURATION/HTTP%20Transparent%20proxy%20%26%20Web%20filtering/)

Configure OPNsense as a transparent HTTP proxy using Squid, with web content filtering via SquidGuard. No client-side configuration required — traffic is intercepted at the firewall level.

### 3\. [High Availability with CARP & pfSync](OPNSENSE%20CONFIGURATION/High%20Availabilty%20with%20CARP%20%26%20pfSync/)

Set up a fault-tolerant OPNsense cluster using **CARP** (Common Address Redundancy Protocol) and **pfSync** for state table synchronization. Includes virtual IP configuration and failover testing.

### 4\. [Suricata IDS/IPS Installation](OPNSENSE%20CONFIGURATION/SURICATA%20Installation%20/)

Deploy **Suricata** as an Intrusion Detection and Prevention System (IDS/IPS) within OPNsense. Covers rule sets, interface binding, alerting, and inline prevention mode.

* * *

## 🛠️ Lab Environment

| Component | Details |
| --- | --- |
| Hypervisor | QEMU/KVM + Virt-Manager |
| Firewall OS | OPNsense (latest stable) |
| Host OS | Linux |

* * *

## 🤔 Why OPNsense?

OPNsense is a fork of pfSense (which itself is a fork of m0n0wall), but distinguishes itself through:

- ✅ Weekly security updates
- ✅ Bi-annual major releases on a predictable schedule
- ✅ Strictly open-source (BSD 2-Clause license)
- ✅ Clean, modern web UI
- ✅ Built-in 2FA/MFA support
- ✅ Active plugin ecosystem

If you're evaluating pfSense vs OPNsense, the honest answer is: it depends on your priorities. OPNsense leans into transparency and open-source philosophy. pfSense has a larger legacy community and tighter integration with Netgate hardware.

* * *

## 🚀 Getting Started

Clone the repo and navigate to the guide you need:

```bash
git clone https://github.com/M-o-j-o/OPNsense.git
cd OPNsense
```

Each folder contains its own documentation. Start with **Installation and Initial Setup** if you're new to OPNsense.

* * *

## 📌 Prerequisites

- A Linux host with KVM/QEMU and Virt-Manager installed
- OPNsense ISO (download from [opnsense.org](https://opnsense.org/download/))
- Basic familiarity with networking concepts (VLANs, subnets, firewall rules)

* * *

## 🤝 Contributing

Found an error or want to add a guide? Feel free to open a pull request or raise an issue. All contributions are welcome.

* * *

## 📄 License

This documentation is provided as-is for educational purposes. OPNsense® is a registered trademark of Deciso B.V.