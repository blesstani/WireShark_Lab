# 🔎 Wireshark Network Traffic Analysis Lab

**Practical Network Forensics • TCP Analysis • Port Scanning Detection**

A hands-on lab exploring how Wireshark can be used to inspect TCP traffic, identify open and closed ports, and recognize SYN-based port scanning.

![Wireshark](https://img.shields.io/badge/Tool-Wireshark-1679A7?style=for-the-badge&logo=wireshark&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-2E8B57?style=for-the-badge)
![Attack](https://img.shields.io/badge/Attack-SYN_Scan-D93025?style=for-the-badge)
![Protocol](https://img.shields.io/badge/Protocol-TCP-F4B400?style=for-the-badge)

## 🎯 Lab Objectives

- 🔵 Filter and inspect TCP SYN packets.
- 🟠 Identify a host sending an unusually high number of connection requests.
- 🟢 Determine which destination ports respond as open or closed.
- 🔴 Recognize the packet pattern associated with a TCP SYN scan.

## 🧪 Lab Scenario

The packet capture contained TCP traffic between:

| Role | IP address |
|------|------------|
| 🔴 Suspected scanning host | `192.168.10.66` |
| 🟢 Target server | `192.168.10.20` |

The investigation focused on repeated SYN requests from the suspected scanning host to the target server.

## 🛠️ Wireshark Filters Used

### ![1](https://img.shields.io/badge/Filter_1-All_SYN_Packets-1679A7?style=for-the-badge)

```diff
tcp.flags.syn == 1 
```

### ![2](https://img.shields.io/badge/Filter_2-Initial_SYN_Requests-F57C00?style=for-the-badge)

Isolate initial SYN requests from the suspected host to the server.

```diff
ip.src == 192.168.10.66 && 
ip.dst == 192.168.10.20 && 
tcp.flags.syn == 1 && 
tcp.flags.ack == 0 
```

### ![3](https://img.shields.io/badge/Filter_3-SYN--ACK_Open_Ports-2E8B57?style=for-the-badge)

```diff
+ ip.src == 192.168.10.20 &&
+ ip.dst == 192.168.10.66 &&
+ tcp.flags.syn == 1 &&
+ tcp.flags.ack == 1
```

> 🟢 These responses indicate ports that accepted the initial connection request.

### ![4](https://img.shields.io/badge/Filter_4-RST_Closed_Ports-D93025?style=for-the-badge)

```diff
- ip.src == 192.168.10.20 &&
- ip.dst == 192.168.10.66 &&
- tcp.flags.reset == 1
```

> 🔴 A reset response indicates that the attempted TCP connection was refused. In this lab, closed ports responded with `RST, ACK`.

## 📡 Key Findings

| Observation | Meaning |
|-------------|---------|
| 🟠 Repeated SYN packets from `192.168.10.66` | Host performing connection probes |
| 🟢 `SYN, ACK` from the server | The destination TCP port is **open** |
| 🔴 `RST, ACK` from the server | The destination TCP port is **closed** |
| 🔵 Requests targeting multiple destination ports | Port-scanning behaviour |

## 🚨 Attack Identified

![SYN Scan](https://img.shields.io/badge/TCP_SYN_Port_Scan-Half--Open_Scan-D93025?style=for-the-badge)

The traffic pattern is consistent with a TCP SYN scan. The scanning host sends SYN packets to multiple ports and uses the server's responses to determine which ports are open or closed.

A SYN scan generally does not complete the normal TCP three-way handshake. This is reconnaissance activity; it is different from a SYN flood, which attempts to exhaust a server's resources.

## 🧠 What I Practised

- Applying Wireshark display filters.
- Reading TCP flags and understanding the three-way handshake.
- Following source and destination IP addresses and ports.
- Distinguishing open-port and closed-port responses.
- Recognizing network reconnaissance in a packet capture.
