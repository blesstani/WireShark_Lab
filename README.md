# 🔎 Wireshark Network Traffic Analysis Lab

**Practical Network Forensics • TCP Analysis • Credential Sniffing • Brute Force Detection • VoIP Investigation**

A hands-on lab exploring how Wireshark can be used to inspect TCP traffic, reconstruct streams, expose plaintext credentials, detect brute force attacks, recognize SYN-based port scanning, and investigate a VoIP scam call.

![Wireshark](https://img.shields.io/badge/Tool-Wireshark-1679A7?style=for-the-badge&logo=wireshark&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-2E8B57?style=for-the-badge)
![Attack](https://img.shields.io/badge/Attack-SYN_Scan-D93025?style=for-the-badge)
![Attack](https://img.shields.io/badge/Attack-Brute_Force-D93025?style=for-the-badge)
![Risk](https://img.shields.io/badge/Risk-Plaintext_Credentials-F57C00?style=for-the-badge)
![Protocol](https://img.shields.io/badge/Protocol-TCP-F4B400?style=for-the-badge)
![Protocol](https://img.shields.io/badge/Protocol-HTTP-8E24AA?style=for-the-badge)
![Protocol](https://img.shields.io/badge/Protocol-SIP%2FRTP-00897B?style=for-the-badge)

## 🎯 Lab Objectives

- 🔵 Filter and inspect TCP SYN packets.
- 🟠 Identify a host sending an unusually high number of connection requests.
- 🟢 Determine which destination ports respond as open or closed.
- 🔴 Recognize the packet pattern associated with a TCP SYN scan.
- 🟣 Follow a TCP stream to reconstruct a full conversation.
- 🟡 Locate credentials sent over HTTP in plain text.
- 🔴 Catch a brute force attack by spotting repeated login attempts.
- 🟦 Play back a captured VoIP call and identify the scammer.

## 🧪 Lab Scenario

The packet capture contained TCP traffic between:

| Role | IP address |
|------|------------|
| 🔴 Suspected scanning host | `192.168.10.66` |
| 🟢 Target server | `192.168.10.20` |

The investigation focused on repeated SYN requests from the suspected scanning host to the target server. Additional captures were then analysed for plaintext credentials, brute force activity, and a suspicious VoIP call.

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

### ![5](https://img.shields.io/badge/Filter_5-Follow_TCP_Stream-8E24AA?style=for-the-badge)

Isolate a single conversation by its stream index:

```diff
# tcp.stream eq <N>
```

Or right-click a packet → **Follow → TCP Stream** (`Ctrl+Alt+Shift+T`).

> 🟣 Client data is shown in red, server data in blue. Stream reassembly turns scattered packets into a readable conversation.

### ![6](https://img.shields.io/badge/Filter_6-HTTP_Plaintext_Credentials-F9A825?style=for-the-badge)

```diff
! http.request.method == "POST"
! http contains "password"
! urlencoded-form
! http.authbasic
```

> 🟡 HTTP sends form data in the clear. Credentials appear in the POST body (`urlencoded-form`) or, for Basic auth, as base64 in the `Authorization` header, which is encoding, not encryption.

### ![7](https://img.shields.io/badge/Filter_7-Brute_Force_Detection-D93025?style=for-the-badge)

```diff
- ip.src == <attacker_ip> && http.request.method == "POST" && http.request.uri contains "login"
- ftp.request.command == "PASS"
- ftp.response.code == 530
+ ftp.response.code == 230
```

Supporting views: **Statistics → Conversations** and **Statistics → I/O Graph** to expose the spike in attempts.

> 🔴 Many failed logins (`530`, `401`, or repeated login POSTs) from one source in a short window, followed by a single success (`230`, `200`/`302`), indicates a brute force attack.

### ![8](https://img.shields.io/badge/Filter_8-VoIP_SIP_%2F_RTP-00897B?style=for-the-badge)

```diff
+ sip
+ sip.Method == "INVITE"
+ rtp
```

Then open **Telephony → VoIP Calls → select the call → Play Streams**.

> 🟦 SIP handles call setup (caller, callee, source IP); RTP carries the actual audio, which Wireshark can decode and play back.

## 📡 Key Findings

| Observation | Meaning |
|-------------|---------|
| 🟠 Repeated SYN packets from `192.168.10.66` | Host performing connection probes |
| 🟢 `SYN, ACK` from the server | The destination TCP port is **open** |
| 🔴 `RST, ACK` from the server | The destination TCP port is **closed** |
| 🔵 Requests targeting multiple destination ports | Port-scanning behaviour |
| 🟣 Followed TCP stream | Full request/response conversation readable in one view |
| 🟡 Credentials visible in HTTP POST body | Data sent unencrypted; anyone on the path can read it |
| 🔴 Many failed logins from one source, then a success | Brute force attack succeeded |
| 🟦 SIP `INVITE` + RTP audio stream | Call metadata and audio recoverable from the capture |
| 🕵️ Caller identity and call content | Scammer identified: `<caller number / IP>` |

## 🚨 Attacks Identified

![SYN Scan](https://img.shields.io/badge/TCP_SYN_Port_Scan-Half--Open_Scan-D93025?style=for-the-badge)
![Brute Force](https://img.shields.io/badge/Brute_Force-Repeated_Login_Attempts-D93025?style=for-the-badge)
![Plaintext](https://img.shields.io/badge/Exposure-HTTP_Plaintext_Credentials-F57C00?style=for-the-badge)
![Scam](https://img.shields.io/badge/Fraud-VoIP_Scam_Call-8E24AA?style=for-the-badge)

### 🔴 TCP SYN Port Scan

The traffic pattern is consistent with a TCP SYN scan. The scanning host sends SYN packets to multiple ports and uses the server's responses to determine which ports are open or closed.

A SYN scan generally does not complete the normal TCP three-way handshake. This is reconnaissance activity; it is different from a SYN flood, which attempts to exhaust a server's resources.

### 🟡 Plaintext Credential Exposure

Login credentials were transmitted over unencrypted HTTP and recovered directly from the capture. **Mitigation:** enforce HTTPS/TLS and never send credentials over HTTP.

### 🔴 Brute Force Attack

A single source sent a rapid series of login attempts with different credentials. The failure-then-success pattern shows the attacker eventually guessed valid credentials. **Mitigation:** rate limiting, account lockout, MFA, and strong passwords.

### 🟦 VoIP Scam Call

The SIP signalling revealed the caller and call path, and the RTP stream was played back to confirm the scam content. The scammer was identified . **Mitigation:** call screening,Implement SIP over TLS and SRTP to encrypt call signaling and voice traffic, blocking unknown numbers, and user awareness.

## 🧠 What I Practised

- Applying Wireshark display filters.
- Reading TCP flags and understanding the three-way handshake.
- Following source and destination IP addresses and ports.
- Distinguishing open-port and closed-port responses.
- Recognizing network reconnaissance in a packet capture.
- Following TCP streams to reconstruct conversations.
- Finding plaintext credentials in HTTP traffic.
- Detecting brute force attacks from login attempt patterns.
- Analysing SIP/RTP traffic and playing back a VoIP call.
- Attributing malicious activity to a source from packet evidence.
