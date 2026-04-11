Redeemer Writeup

Target IP: 10.129.71.57
Difficulty: Starting Point - Tier 0
Author: crash.99
Date: April 11, 2026
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Table of Contents

    Executive Summary
    Toolbox: The Modern Recon Stack
    Deep Dive: Understanding Redis
    Phase 1: Enumeration (Scanning)
    Phase 2: Exploitation (Database Interaction)
    Findings & Flag
    Remediation
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**Executive Summary**

Redeemer is an entry-level machine on Hack The Box designed to test a researcher's ability to identify and interact with misconfigured NoSQL databases. The primary vulnerability discovered was an unauthenticated Redis database service. By leveraging modern scanning tools and a direct connection to the service, we were able to extract the hidden flag without requiring any administrative credentials.

**Toolbox: The Modern Recon Stack**

In this engagement, we prioritized speed and efficiency by utilizing modern alternatives to traditional tools.
1. RustScan
While Nmap is the industry standard, RustScan is built for speed. It is written in Rust and can scan all 65,535 ports in a matter of seconds.

    Why we used it: To quickly identify open ports across the entire range before focusing on specific services.

    Command: rustscan -a 10.129.71.57 -- -sV

2. Naabu
Naabu is another high-speed port scanner (by ProjectDiscovery) that excels in reliability. It is often used in bug bounty hunting because it handles packet loss and network congestion gracefully.

4. Redis-Cli
The native command-line interface used to send commands directly to the Redis server.
Understanding Redis

**What is Redis?**

Redis (Remote Dictionary Server) is an open-source, in-memory data structure store. Unlike traditional databases (like MySQL) that store data on a disk, Redis keeps data in RAM for lightning-fast access.
Database Type: NoSQL (Key-Value)

Redis is a Key-Value database. Think of it like a giant dictionary where every piece of information has a "name" (the Key) and the "information" itself (the Value).

    Speed: Capable of millions of requests per second.

    Persistence: Though it's in-memory, it can periodically save data to disk.

    Security Risk: By default, older versions of Redis do not require a password. If exposed to the internet, anyone can read, modify, or delete the data.

**Phase 1: Enumeration (Scanning)**
Our initial scan revealed only one point of entry: Port 6379.
<img width="1920" height="937" alt="Screenshot_2026-04-11_09_00_40" src="https://github.com/user-attachments/assets/ef1475a8-8f6a-47c2-9c45-44c729976825" />
The version 5.0.7 confirmed we were dealing with a Redis instance. The "Reason: syn-ack" indicates the port is wide open and ready for a connection.

**Phase 2: Exploitation (Database Interaction)**
**1. Establishing Connection**
Using redis-cli, we connected to the target without providing a password, confirming the "Unauthenticated Access" vulnerability.
<img width="1920" height="937" alt="Screenshot_2026-04-11_09_03_00" src="https://github.com/user-attachments/assets/6cae860f-5340-4591-8d86-80fab92ff7e8" />

**2. Server Enumeration**

Once inside, I queried the server for information about the stored data.

    info: Showed the server was running on Linux and had one active database (db0) containing 4 keys.
**3. Key Discovery**

To see what was inside the database, I listed all available keys:
<img width="1920" height="937" alt="Screenshot_2026-04-11_09_09_39" src="https://github.com/user-attachments/assets/a8c88118-1ed8-4c5f-b628-a7dc0ed3fcc1" />
4. Data Exfiltration

The flag key was the obvious target. Using the get command, I retrieved the value:
<img width="1920" height="937" alt="Screenshot_2026-04-11_09_11_22" src="https://github.com/user-attachments/assets/7f04c4e8-f772-4d2b-923a-ed42eb9172e1" />

Findings & Flag

    Vulnerability: Unauthenticated Redis Access

    Exploit Vector: Direct CLI connection via Port 6379

**Remediation**

To secure this machine, the following steps should be taken:

    Enable Authentication: Set a strong password in the redis.conf file using the requirepass directive.

    Bind to Localhost: Ensure Redis only listens on 127.0.0.1 unless external access is strictly necessary.

    Change Default Port: Moving the service from 6379 to a non-standard port can help hide it from automated "low-hanging fruit" scans.

    Firewall Rules: Use ufw or iptables to restrict access to port 6379 to only trusted IP addresses.
  
  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  state = success 
  Exfiltrated Flag: 03e1d2b376c37ab3f5319922053953eb

    
