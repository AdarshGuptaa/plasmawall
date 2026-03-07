**Linux Firewall from scratch**

**Stateless vs Stateful firewalls**
_**Stateless firewalls** blindly follow the rules of the hash table and do not remember outgone requests and are susceptible to masked attacks.
**Stateful firewalls** are context aware and remember things and infer through them to prevent attacks._



1\. **Kernel Space: Packet Interception:**
- Intercept a packet.
- Unpack its details and print it to the kernel log(dmesg)
- Netfilter Hooks



2\.** User Space: CLI tool for rule management of packets:**
- Create a HashMap to store rules for packets.
- Move them from user space to kernel space.
- Netlink Sockets or a character device interface



3\. **Parse Packet's header against HashMap of rules:**
- Parsing: Extract: Ethernet Header| IP Header| Transport Layer Header
- Matching: Match parser headers in the Map

4\. **Packet Execution:**
- NF\_ACCEPT: Continue transmission
- NF\_DROP: Discard
- NF\_QUEUE: Send to User Space for further inspection

_A "connection" is purely a logical concept—it is a shared agreement and shared memory between two computers._
_Ex: Establishing connection between you and a website purely for easier authentication of subsequent packet deliveries._

5\. **Connection Tracking: (Conversation tracker)**

- To prevent random access and connections to the network this acts as a very smart Bouncer
- Identifies conversations using a sort of fingerprint for each conversation = 5 tuples = src IP| Dest IP| Src Port| Dest Port| Protocol
- Stores this conversation fingerprint for new conversations
- Allows free communication for established conversations
- Closes conversations

**Life cycle of a packet through the firewall:**
_**Packet Arrives at a socket -> Packet is parsed of its headers -> Some stateless rules are applied (Could eject packet) -> Stateful Engine works (conversation tracking) -> If new connection then stateless fallback matches the new connection through a HashMap of rules**_

**Types of attacks and how to protect:**

**SYN Floods (A type of DDoS)**
**The Attack**: An attacker sends thousands of TCP SYN (hello) packets to your server but never completes the handshake with the final ACK. Your server allocates memory for each "half-open" connection until it runs out of RAM and crashes.

**The Defense**: Your stateful firewall tracks these half-open connections. You can configure it to enforce Rate Limiting (e.g., "Only allow 20 new SYN packets per second from a single IP"). It drops the excess, saving your server's memory.

2\. **Port Scanning**
**The Attack**: Attackers use tools like Nmap to systematically ping every port on your IP address to see what services (SSH, HTTP, FTP) are running and vulnerable.

**The Defense**: A "default deny" firewall policy. If a port isn't explicitly open in your stateless rules, the firewall silently DROPs the packet instead of sending a polite "Connection Refused" (REJECT) message. If the attacker gets no response, it slows them down and obscures your system's architecture.

3\. IP Spoofing
**The Attack**: An attacker modifies the packet header so the "Source IP" looks like it's coming from a trusted internal machine (e.g., 192.168.1.50) instead of the outside internet, hoping to bypass your rules.

**The Defense**: Ingress/Egress Filtering. You write a strict stateless rule: "If a packet arrives on the external network interface, but claims to have a source IP from my internal subnet, drop it immediately."

***HOW TO BUILD USING RUST:
eBPF using AYA framework in RUST*

Prerequisites for dev:**

* Set up a Virtual Machine using QEMU(Quick Emulator) and KVM(Kernel-based Virtual Machine)
* Choose a Kernel 6.1 or newer kernel Distro: Rust Support came and Kernel changes wont break for multiple versions of 6.xxxx
* Toolchain requirements: Rust **Nightly,** LLVM \& Clang, bpf-linker, bpf-tool

**Crates to be used:**

*Kernel Space and User Space Requirements are separate because of their inherent nature*

***Kernel Space:							User Space:***

*|Restricted							|Everything allowed*

*|Only core functionality enabled*

*|No standard Lib ex: Vec Stack Map etc disabled*



<b>Kernel Space:</b>

1. aya-ebpf: Interact with hooks and establish the HashMap
2. network-types: Network Headers converted from c to rust typings safely
3. aya-log-ebpf: Logging stuff directly from kernel to user-space

**User Space:**

1. aya
2. tokio: 
3. clap: To create CLI commands
4. anyhow: Error Handling Crate
5. aya-log: Catches logs from aya-log-ebpf

