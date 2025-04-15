---
theme: white # Or your preferred theme
revealjs:
  width: 960
  height: 700
  margin: 0.1 # Adjust margin as needed
  center: true
---

# Day 3: IPv6 Management & Security

*Planning, Monitoring, Troubleshooting, Securing*

<aside class="notes">
Welcome to Day 3, our final day focusing on IPv6 Essentials. Yesterday, we explored migration and coexistence strategies like Dual-Stack, Tunneling, and Translation, and you practiced configuring NLB ingress and public dual-stack subnets. Today, we shift our focus to the ongoing operational aspects: How do we effectively manage IPv6 networks, and what are the key security considerations? We'll cover address planning, monitoring, troubleshooting, DNS/DHCPv6 management, and security best practices, followed by labs on GRE tunneling and firewall configuration.
</aside>

---

## Day 3 Agenda

* **Recap:** Day 2 Key Concepts & Labs 3 & 4 Achievements
* **Module 7:** IPv6 Address Management (IPAM)
* **Module 8:** Monitoring & Troubleshooting IPv6
* **Quiz 5**
* **Lab 5:** IPv6-over-IPv4 Tunneling (GRE) *(Morning)*
* **Lunch Break**
* **Module 9:** DNS & DHCPv6 Management Considerations
* **Module 10:** IPv6 Security
* **Quiz 6**
* **Lab 6:** IPv6 Security Configuration (NACLs & SGs) *(Afternoon)*
* **Course Wrap-up & Q/A**

<aside class="notes">
Here's the plan for today. We'll start with a recap of Day 2's labs. Then, we'll dive into management topics: planning and managing the vast IPv6 address space (IPAM), followed by monitoring techniques and common troubleshooting scenarios specific to IPv6. After a quiz, you'll get hands-on with Lab 5, manually configuring a GRE tunnel. Following lunch, we'll discuss DNS and DHCPv6 management choices. The main focus of the afternoon lecture will be IPv6 Security – understanding the threats and mitigation techniques, including firewalls. After a final quiz, you'll apply these security concepts in Lab 6 by configuring AWS Network ACLs and Security Groups for IPv6 traffic. We'll conclude with a course wrap-up and final Q&A.
</aside>

---

## Recap: Day 2 Achievements

* **Lectures Covered:** Dual-Stack, Tunneling (GRE, Automatic), Translation (NAT64/DNS64, NLB Ingress).
* **Lab 3 Completed:** Configured NLB to provide ingress connectivity to private IPv6-only instances.
* **Lab 4 Completed:** Configured and tested public dual-stack subnet and instance using IGW (Challenge Lab).
    * *(Optional Extension): Configured and tested private dual-stack subnet using NGW/EIGW.*

<aside class="notes">
Quick recap of Day 2: We discussed the core migration strategies in detail. In the labs, you successfully used a Network Load Balancer to provide inbound access to your IPv6-only services and tackled the challenge lab to set up and test direct internet access for a public dual-stack instance via the IGW. Some of you may have also completed the optional extension configuring a private dual-stack instance. This means you've now seen and configured IPv4-only private, IPv6-only private, public dual-stack, and potentially private dual-stack environments within the Service Provider VPC! Today we focus on operating and securing these environments.
</aside>


---

## Module 7: IPv6 Address Management (IPAM)

<aside class="notes">
Managing IPv4 addresses was often about conserving a scarce resource. Managing IPv6 is different – it's about structured planning to leverage the abundance of addresses effectively.
</aside>

---

### Why IPAM for IPv6?

* **Abundance vs. Scarcity:** Not about *saving* addresses, but *organizing* them logically.
* **Complexity:** Manual tracking of 128-bit hex addresses across potentially thousands of /64 subnets is impractical and error-prone.
* **Goals:**
    * **Structure:** Support hierarchical routing aggregation.
    * **Scalability:** Easily accommodate future growth.
    * **Manageability:** Simplify allocation, tracking, troubleshooting, security policy definition.
    * **Automation:** Enable network automation tools to interact with address space.

<aside class="notes">
Why do we need a formal IP Address Management (IPAM) strategy for IPv6 when addresses are so plentiful? Precisely *because* they are plentiful. Trying to manage potentially trillions upon trillions of addresses per site without a plan leads to chaos. Good IPv6 IPAM isn't about conserving addresses like in IPv4; it's about creating a logical, hierarchical structure. This structure simplifies routing (allowing for route summarization), makes it easy to add new networks or sites later, simplifies troubleshooting (knowing where an address *should* be located), makes writing firewall rules easier (applying rules to logical blocks), and provides a database for automation tools. Manual tracking with spreadsheets just doesn't scale for IPv6.
</aside>

---

### Hierarchical Planning Approach

* **Top-Down Allocation:**
    * **ISP/RIR Allocation:** Typically `/48` or `/56` assigned to the organization.
    * **Site Allocation:** Divide the main prefix into smaller prefixes for distinct physical sites or large logical zones (e.g., assign `/56`s or `/60`s out of a `/48`).
    * **Subnet Allocation:** Divide site prefixes into `/64` prefixes for individual VLANs/LAN segments. **Always use /64 for end-user subnets.**
    * **Interface ID:** Last 64 bits assigned via SLAAC, DHCPv6, or statically.

[DIAGRAM PLACEHOLDER: Tree diagram showing /48 -> /56 (Sites) -> /64 (Subnets)]

<aside class="notes">
The standard approach is hierarchical. You start with the large block assigned to your organization by your ISP or RIR – commonly a /48 or /56. You then carve that up logically. Assign slightly smaller blocks (like /56s or /60s) to major physical locations (HQ, Data Center, Branch Office) or significant logical zones (Production, Dev, DMZ). Within each site/zone prefix, you then allocate /64 prefixes for each individual network segment or VLAN. It is crucial to stick to /64 for end-user subnets, as many IPv6 features like SLAAC depend on it. The final 64 bits, the Interface ID, are then assigned to hosts within that /64 subnet using methods like SLAAC, DHCPv6, or static configuration.
</aside>

---

### Prefix Allocation Best Practices

* **/64 for LANs:** **Strongly recommended.** Don't try to conserve by using smaller prefixes like /112 or /120 on LAN segments – it breaks SLAAC and other features. Treat a /64 as the smallest practical unit for assignment.
* **ISP Allocation Size:** Aim for at least a `/48` if possible, especially for medium-to-large organizations, to allow ample room for subnetting across multiple sites. A `/56` is common for smaller sites or home users.
* **Nibble Boundaries:** When subnetting (dividing larger prefixes), using "nibble boundaries" (dividing along 4-bit increments, corresponding to one hex digit) makes addresses easier to read and manage.
    * `/48` -> `/52` -> `/56` -> `/60` -> `/64`
* **Document Everything:** Use an IPAM tool or at least a structured documentation system.

<aside class="notes">
Some key best practices: First and foremost, always use a /64 prefix length for your LAN segments where end hosts connect. Don't be tempted by IPv4 habits to use smaller prefixes to save addresses – you have plenty, and using anything other than /64 breaks critical features like SLAAC. When getting a prefix from your ISP, ask for a /48 if you're a reasonably sized organization; this gives you 65,536 /64 subnets to work with. A /56 (giving 256 /64s) is often sufficient for smaller sites. When you subdivide your allocated prefix, try to align your subnet boundaries on 4-bit increments (nibbles), as this keeps the hexadecimal digits tidy. For example, breaking a /48 into /52s, then /56s, then /60s, then /64s. And critically, document your plan and allocations meticulously, preferably using a dedicated IPAM tool.
</aside>

---

### Subnetting Strategies & Documentation

* **Logical Grouping:** Assign subnet IDs based on function, location, or type (e.g., `2001:db8:SITE:SUBTYPE::/64`).
    * Example: `SITE` = `0010` (HQ), `SUBTYPE` = `0001` (Servers), `0002` (Users), `0003` (VoIP).
    * Result: `2001:db8:10:1::/64` (HQ Servers), `2001:db8:10:2::/64` (HQ Users).
* **Simplicity:** Keep the plan as simple as possible while allowing for growth.
* **Documentation / IPAM Tool:** Essential for tracking:
    * Assigned prefixes (VPC, Site, Subnet levels)
    * Subnet details (VLAN ID, Description, Location)
    * Static address assignments (Servers, Routers)
    * DHCPv6 pool ranges (if used)

<aside class="notes">
How do you assign the Subnet IDs within your site prefix? Plan it logically. Embed meaning into the subnet portion of the address if possible. For example, if your site has prefix `2001:db8:10::/56`, you have the 4th hextet (from `00` to `ff`) for subnetting into /64s. You could assign `2001:db8:10:1::/64` for servers, `2001:db8:10:2::/64` for user VLAN A, `2001:db8:10:3::/64` for user VLAN B, `2001:db8:10:F::/64` for printers, etc. Keep it consistent and document it rigorously in your IPAM system. Track which prefixes are allocated, what each /64 subnet is used for (VLAN, description, location), any important static addresses assigned within those subnets, and any DHCPv6 ranges.
</aside>

---

### IPAM Tools Overview

* **Why Needed:** Spreadsheets don't scale, lack validation, hard to integrate.
* **Features:**
    * Hierarchical prefix/address management.
    * IPv4 & IPv6 support.
    * Subnet scanning / discovery (optional).
    * Tracking utilization.
    * Role-based access control.
    * API for automation.
    * Integration with DNS/DHCP.
* **Examples:**
    * Open Source: phpIPAM, NetBox, GestióIP
    * Commercial: Infoblox, BlueCat, SolarWinds IPAM, EfficientIP

<aside class="notes">
Manual tracking via spreadsheets quickly becomes unmanageable with IPv6. Dedicated IPAM tools are highly recommended. Good IPAM tools provide a structured database for your address space, allowing you to define your hierarchy, allocate prefixes and subnets, track individual address assignments (especially static ones), document usage, manage utilization, control access, and often integrate with DNS and DHCP systems. They provide an API that network automation tools can use to request or update address information. There are excellent open-source options like phpIPAM and NetBox, as well as established commercial vendors like Infoblox and BlueCat. Using an IPAM tool is a cornerstone of effective IPv6 management.
</aside>


---

## Module 8: Monitoring & Troubleshooting IPv6

<aside class="notes">
Once IPv6 is deployed, how do we monitor its health and troubleshoot problems when they arise? Many familiar concepts apply, but there are IPv6-specific tools and issues.
</aside>

---

### Monitoring Goals & Basic Tools

* **Goals:** Ensure reachability, measure performance (latency/loss), detect failures, identify security issues, capacity planning.
* **Basic Reachability:**
    * `ping6` (ICMPv6 Echo Request/Reply): Verifies L3 connectivity, basic latency/loss. Use with GUAs, LLAs (requires scope ID `%interface`), Loopback (`::1`).
* **Path Discovery:**
    * `traceroute6` (or `tracert -6` on Windows): Sends packets with increasing Hop Limits to map the L3 path to a destination. Uses UDP or ICMPv6.

<aside class="notes">
Monitoring goals are similar to IPv4: Is it working? How well? Are there problems? Security threats? Do we need more capacity? For basic reachability, `ping6` is the direct equivalent of `ping`. It sends ICMPv6 Echo Requests. Remember you can ping Global addresses, Link-Local addresses (but you usually need to specify the outgoing interface using `%eth0` or similar), and the loopback `::1`. To see the path packets take, use `traceroute6` (or `tracert -6` on Windows). It works similarly to IPv4 traceroute by sending packets with incrementing Hop Limits and listening for ICMPv6 'Time Exceeded' messages from routers along the path.
</aside>

---

### Key Troubleshooting Tools (Linux)

* **IP Configuration:**
    * `ip addr show` or `ip -6 addr show`: Displays interface IP addresses (LLA, GUA), states (tentative, preferred).
    * `ip -6 route show`: Displays the IPv6 routing table.
    * `ip -6 neighbor show`: Displays the Neighbor Cache (NDP equivalent of ARP cache - maps IPv6 to MAC addresses, shows reachability state like STALE, REACHABLE, DELAY).
* **Packet Capture:**
    * `tcpdump -i <interface> ip6`: Capture IPv6 traffic. Use filters (e.g., `icmp6`, `host ::1`, `port 80`). `-n` prevents name resolution, `-v`/`-vv` increases verbosity.
    * `tshark`: Command-line Wireshark, provides detailed decoding (as seen in optional lab).

<aside class="notes">
On Linux systems, the `ip` command from the `iproute2` package is your primary tool. `ip -6 addr show` displays IPv6 addresses on interfaces and their status. `ip -6 route show` shows the kernel's IPv6 routing table – look for your default route (`::/0`) and specific subnet routes. `ip -6 neighbor show` is critical for troubleshooting local connectivity; it shows the Neighbor Cache built by NDP, mapping neighbor IPv6 addresses to their MAC addresses and indicating their reachability state (STALE means needs verification, REACHABLE is good, DELAY/PROBE means verification is in progress). And of course, packet capture using `tcpdump` (filtering on `ip6` or `icmp6`) or `tshark` is invaluable for deep analysis.
</aside>

---

### Common Issue: Path MTU Discovery (PMTUD)

* **IPv6 Requirement:** IPv6 routers **do not** fragment packets. Only the source host can fragment.
* **PMTUD Goal:** Source host discovers the smallest MTU along the entire path to the destination to avoid sending packets that are too large.
* **Mechanism:**
    1. Host sends large packet (e.g., 1500 bytes).
    2. Router encounters link with smaller MTU (e.g., 1400 bytes).
    3. Router **drops** the large packet.
    4. Router sends **ICMPv6 "Packet Too Big"** message back to source, specifying the smaller MTU (1400).
    5. Source host reduces packet size for that destination and resends.
* **Problem:** Firewalls often incorrectly **block all ICMPv6**, including "Packet Too Big" messages.
* **Symptom:** Small packets (like ping) work, but large transfers (HTTP, SCP, etc.) hang or fail mysteriously.
* **Fix:** Ensure firewalls (NACLs, SGs, external) **allow** ICMPv6 Type 2 ("Packet Too Big") messages inbound to your hosts.

<aside class="notes">
A very common and frustrating issue in IPv6 involves Path MTU Discovery. Unlike IPv4, IPv6 routers are forbidden from fragmenting packets in transit. If a packet is too big for a link, the router drops it and sends an ICMPv6 'Packet Too Big' message back to the original source host, telling it the maximum size allowed on that link. The source host is then responsible for reducing its packet size for that specific destination. This PMTUD process is essential. The problem arises when firewalls, often configured with IPv4 mindsets, block *all* ICMP traffic, including the necessary ICMPv6 'Packet Too Big' messages. If the source never receives this message, it keeps sending large packets that get dropped, leading to connections stalling or failing, often only for large data transfers while small pings work fine. The fix is crucial: configure your firewalls (NACLs, Security Groups, on-prem firewalls) to permit inbound ICMPv6 Type 2 messages.
</aside>

---

### Common Issue: Neighbor Discovery (NDP)

* **NDP Functions:** Address resolution (NS/NA), Router discovery (RS/RA), DAD, NUD.
* **Potential Problems:**
    * **Missing Router Advertisements (RAs):** Hosts won't get default gateway or prefix info for SLAAC. Check router config.
    * **Incorrect RA Flags:** M/O flags control DHCPv6 usage. Incorrect flags lead to wrong addressing method.
    * **Neighbor Cache Issues:** `ip -6 neigh` shows `FAILED` or stays `STALE`. Could be L2 issue, firewall blocking NS/NA (ICMPv6 Types 135/136), or neighbor actually down.
    * **Duplicate Address Detection (DAD) Failures:** Host detects its tentative address is already in use (sees NA for its NS). Check for static misconfiguration or rogue devices.
* **Troubleshooting:** Check router RA config, check NDP cache (`ip -6 neigh`), capture ICMPv6 traffic (`tcpdump -i eth0 icmp6`).

<aside class="notes">
Neighbor Discovery is fundamental for IPv6 local link operation, so problems here cause significant issues. If hosts aren't getting addresses via SLAAC or can't find their default gateway, check if the local router is actually sending Router Advertisements (RAs) and if they contain the correct prefix information (Prefix Information Option with 'A' flag set) and router lifetime. Check the RA flags (Managed/Other) if using DHCPv6 alongside SLAAC. If hosts can't reach local neighbors, check the neighbor cache (`ip -6 neigh`). If entries are FAILED or always STALE, investigate Layer 2 connectivity or potential firewalls blocking Neighbor Solicitation/Advertisement messages (ICMPv6 types 135 & 136). DAD failures usually point to an IP address conflict. Use packet captures filtering on `icmp6` to see the NDP messages directly.
</aside>

---

### Monitoring Tools (Brief Overview)

* **SNMP:** SNMPv3 supports IPv6 transport and IPv6-specific MIBs (Management Information Bases) exist for monitoring interface stats, routing tables, etc. Requires SNMP agent configuration on devices.
* **Flow Monitoring (NetFlow/sFlow/IPFIX):** Modern versions support exporting detailed IPv6 flow records (Src/Dst IP, Ports, Protocol, Next Header, bytes, packets) for traffic analysis, capacity planning, security monitoring. Requires exporter config on routers/switches and a flow collector/analyzer.
* **Logging:** Ensure Syslog, firewall logs, application logs capture full IPv6 addresses correctly for incident analysis.

<aside class="notes">
Beyond basic ping/traceroute, robust monitoring uses standard protocols. SNMPv3 works over IPv6 and provides MIBs for querying IPv6 interface counters, routing information, and more. Flow monitoring protocols like NetFlow v9, IPFIX, and sFlow fully support exporting IPv6 flow data, which is invaluable for understanding traffic patterns and volumes – you just need devices configured to export flows and a collector to receive and analyze them. And don't forget logging – ensure your various systems (syslog, firewalls, web servers) are configured to log the full 128-bit IPv6 addresses properly, as truncated logs are useless for troubleshooting.
</aside>

---

### Quiz 5: IPAM, Monitoring & Troubleshooting

<aside class="notes">
Let's take a short break from lectures for Quiz 5, covering the management, monitoring, and troubleshooting topics we just discussed.
(Administer Quiz 5 - ~10 mins)
Any questions on those areas before we start Lab 5?
</aside>


---

### Lab 5: IPv6-over-IPv4 Tunneling (GRE)

*(Morning Lab)*

<aside class="notes">
Now it's time for Lab 5, where you'll get hands-on experience manually configuring a GRE tunnel between two instances to carry IPv6 traffic over an IPv4 path. Please refer to your Lab Guide for the detailed steps.
</aside>


---

### LUNCH BREAK (1 Hour)


---

## Module 9: DNS & DHCPv6 Management Considerations

<aside class="notes">
Welcome back. Let's quickly touch on some management aspects related to DNS and address assignment options (DHCPv6/SLAAC).
</aside>

---

### DNS Recap & Reverse DNS

* **Forward DNS:** Hostname -> IP Address
    * IPv4: `A` records
    * IPv6: `AAAA` ("Quad-A") records
* **Reverse DNS:** IP Address -> Hostname
    * IPv4: `PTR` records in `in-addr.arpa` domain.
    * IPv6: `PTR` records in `ip6.arpa` domain.
* **`ip6.arpa` Structure:** Uses nibbles (4 bits / 1 hex digit) in reverse order.
    * Example: `2001:db8:1:2::abcd` -> Reverse zone is `...2.0.0.0.1.0.0.0.8.b.d.0.1.0.0.2.ip6.arpa` (complex!)
* **Importance:** Reverse DNS is used for logging, troubleshooting, some anti-spam checks. Needs careful management.

<aside class="notes">
Quick DNS recap: AAAA records map hostnames to IPv6 addresses. For reverse lookups (IP to hostname), IPv6 uses PTR records just like IPv4, but the special domain is `ip6.arpa`. Constructing the reverse lookup name is more complex because it reverses the address one hex digit (nibble) at a time. For example, the reverse lookup for `2001:db8:1:2::abcd` involves reversing `d.c.b.a.0.0.0.0.0.0.0.0.0.0.0.0.2.0.0.0.1.0.0.0.8.b.d.0.1.0.0.2` and appending `.ip6.arpa`. Managing these reverse zones accurately is important for logging and troubleshooting, and IPAM tools can often help automate PTR record creation.
</aside>

---

### DNS Resolver Considerations

* **Dual-Stack Resolvers:** DNS servers themselves should be reachable over both IPv4 and IPv6.
* **Client Configuration:** Clients need DNS server addresses for *both* protocols (or rely on mechanisms like NDP RDNSS for IPv6 DNS).
* **DNS64 Interaction:** Remember DNS64 is a function of the *resolver* the client uses, not necessarily the authoritative server for the domain. AWS Route 53 Resolver provides this when enabled.

<aside class="notes">
When managing DNS in a dual-stack world, ensure your recursive DNS resolvers (the servers your clients talk to) are themselves reachable over both IPv4 and IPv6. Clients need to be configured with appropriate resolver addresses for both stacks. This might come from DHCPv4, DHCPv6, or the RDNSS (Recursive DNS Server) option in NDP Router Advertisements for IPv6. Also, recall that DNS64 functionality resides on the resolver; it intercepts queries from IPv6-only clients and synthesizes AAAA records if needed.
</aside>

---

### DHCPv6 vs SLAAC Management Recap

* **SLAAC (Stateless Address Autoconfiguration):**
    * Uses NDP RAs (Prefix Information Option with 'A' flag).
    * Host self-generates Interface ID (EUI-64 or Privacy Extensions).
    * Simple, no server state needed for addresses.
    * **Needs RA Options (RDNSS) or Stateless DHCPv6 for DNS server info.**
* **DHCPv6 (Stateful):**
    * Requires DHCPv6 server infrastructure.
    * Server assigns specific addresses and options (DNS, NTP, etc.).
    * Provides centralized control and logging of address assignments.
    * Triggered by RA M-flag (Managed Address Configuration).
* **DHCPv6 (Stateless):**
    * Host gets address via SLAAC.
    * Host contacts DHCPv6 server *only* for additional options (DNS, NTP, etc.).
    * Triggered by RA O-flag (Other Configuration).

<aside class="notes">
Quick recap on address assignment management. SLAAC is the simplest – hosts listen to RAs and configure their own address using the advertised prefix and a self-generated IID. However, SLAAC alone doesn't traditionally provide DNS server info (though the RDNSS option in RAs can). Stateful DHCPv6 works like IPv4 DHCP, with a server assigning specific addresses and options, giving you tight control but requiring server infrastructure. Stateless DHCPv6 is a hybrid: the host gets its address via SLAAC but contacts a DHCPv6 server just to get other options like DNS servers. The choice depends on the RA flags set by the router: M=1 means use stateful DHCPv6 for addresses; O=1 means use stateless DHCPv6 for options (used with or without M=1); if both are 0, use only SLAAC (and maybe RDNSS). Managing these RA flags correctly across your routers is key.
</aside>

---
---

## Module 10: IPv6 Security

<aside class="notes">
Our final lecture module focuses on Security considerations specific to IPv6. While many principles remain the same, IPv6 introduces new aspects and changes how some existing mechanisms work.
</aside>

---

### Security Mindset Shift

* **No NAT != No Security:** The lack of ubiquitous NAT for address conservation in IPv6 **does not** mean networks are inherently less secure. Security relies on **firewalls**.
* **End-to-End Reachability:** Restored by IPv6, but means every host is potentially reachable. Firewall policies become even more critical.
* **Larger Address Space:** Makes traditional brute-force scanning much harder, but reconnaissance still possible (e.g., scanning known prefixes, DNS enumeration).
* **New Protocols/Features:** NDP, Extension Headers introduce new potential attack vectors that need securing.

<aside class="notes">
Migrating to IPv6 requires a slight shift in security thinking. A common misconception is that NAT provides security in IPv4. While it hides internal topology, NAT is *not* a firewall. True security comes from stateful firewalls, which are just as essential in IPv6. Because IPv6 restores end-to-end reachability (every device can have a unique global address), your firewall policies at the edge and on the host become paramount. The huge address space makes random scanning infeasible, but attackers can still find targets by scanning allocated prefixes or enumerating DNS. We also need to consider security for new protocols like NDP and features like Extension Headers.
</aside>

---

### NDP Security Issues & Mitigations

* **Neighbor Solicitation/Advertisement Spoofing:** Attacker sends fake NA claiming to be the gateway or another host (like ARP spoofing -> Man-in-the-Middle).
* **Router Advertisement (RA) Spoofing:** Attacker sends fake RAs advertising itself as the default gateway, incorrect prefixes, or setting flags to cause DoS.
* **Duplicate Address Detection (DAD) DoS:** Attacker responds to DAD NS messages, preventing legitimate hosts from using addresses.
* **Mitigations:**
    * **RA Guard (Switch Feature):** Filters RAs on switch ports, only allowing legitimate ones from trusted router ports. (Commonly implemented).
    * **SAVI (Source Address Validation Improvement):** Switch tracks valid IP-MAC-Port bindings to prevent spoofing. (Less common).
    * **SEND (Secure Neighbor Discovery - RFC 3971):** Uses cryptographic methods (CGAs, certificates) to secure NDP messages. Powerful but complex to deploy, not widely used.
    * **Basic Port Security:** Still relevant.

<aside class="notes">
Neighbor Discovery Protocol, operating on the local link, is vulnerable. Attackers can spoof Neighbor Advertisements to redirect traffic (like ARP spoofing). More dangerously, they can send rogue Router Advertisements to become the default gateway, hijack traffic, or cause Denial of Service by advertising bad prefixes or flags. They can also interfere with DAD. The most common and practical mitigation deployed today is RA Guard, a switch feature that blocks unauthorized RAs on end-user ports. Other mechanisms like SAVI (validating source addresses) and SEND (cryptographically securing NDP) exist but are less widely deployed due to complexity. Basic switch port security remains important too.
</aside>

---

### IPsec in IPv6

* **Mandatory Support:** Support for IPsec (Authentication Header - AH, Encapsulating Security Payload - ESP) is **mandatory** in the core IPv6 protocol specifications. (Unlike IPv4 where it was optional).
* **Implementation:** While support is mandatory, *use* of IPsec is still optional and requires configuration.
* **Functionality:** Same as IPv4:
    * **AH (Protocol 51):** Provides integrity and authentication for the IP header and payload, but no confidentiality.
    * **ESP (Protocol 50):** Provides confidentiality (encryption) and optionally integrity/authentication for the payload. Can operate in transport or tunnel mode.
* **Usage:** End-to-end security, VPNs (often GRE over IPsec or pure IPsec tunnels).

<aside class="notes">
A key security enhancement is that support for IPsec is baked into the IPv6 standard itself; it's not an optional add-on like it was for IPv4. This means all compliant IPv6 stacks *must* be capable of using IPsec. Of course, actually *using* IPsec still requires configuration. The protocols themselves, AH and ESP, function the same way as in IPv4. AH provides authentication and integrity without encryption, while ESP provides encryption and optional integrity/authentication. They can be used for securing end-to-end communication or building VPN tunnels. The mandatory support provides a stronger baseline security capability for IPv6 networks.
</aside>

---

### Firewalling IPv6

* **Stateful Firewalls Essential:** Just like IPv4, stateful inspection (tracking connections) is the core of effective firewalling.
* **Rule Logic:** Define rules based on IPv6 source/destination addresses, prefixes, L4 ports, and protocol (Next Header value).
* **ICMPv6 Filtering:** **Do NOT block all ICMPv6!** It's crucial for basic operation (NDP, PMTUD, etc.). Allow necessary types (e.g., Echo Req/Rep, Packet Too Big, NDP messages NS/NA/RS/RA) while potentially blocking others (like Redirects from non-routers).
* **Extension Header Filtering:** Firewalls need to be able to parse and filter based on Extension Headers if required. This can be complex and performance-impacting. Some firewalls might drop packets with certain EHs or too many EHs. Policy needs careful consideration.
* **AWS Context:** Security Groups (Stateful, instance-level) and Network ACLs (Stateless, subnet-level) both fully support IPv6 rules alongside IPv4 rules.

<aside class="notes">
Firewalling principles remain similar: stateful inspection is key. You define rules based on source/destination IPv6 addresses (or prefixes), ports, and protocol (identified by the Next Header field). A critical difference is ICMPv6 – it's *not* just for ping like ICMPv4 often is. ICMPv6 carries vital NDP messages and PMTUD messages. Blocking all ICMPv6 will break your network. You need fine-grained rules to allow essential types while blocking potentially harmful ones. Filtering based on IPv6 Extension Headers is another challenge. Ideally, your firewall can inspect and make decisions based on specific EHs (like blocking certain Routing Headers), but this adds complexity and potential performance hits. Some firewalls might just drop packets with unknown or excessive EHs. In AWS, both Security Groups and Network ACLs work just like they do for IPv4, allowing you to define separate rules for IPv6 traffic based on addresses, ports, and protocols. Lab 6 will focus on this.
</aside>

---

### Other Security Considerations

* **Scanning:** While brute-force scanning the entire 128-bit space is impossible, attackers can scan known allocated prefixes, enumerate DNS, or find addresses from logs/leaks. Don't rely on obscurity.
* **Reconnaissance:** Tools can use NDP or ICMPv6 Echo Requests to map active hosts on local links if not filtered.
* **Privacy Addresses (RFC 4941):** Temporary, random Interface IDs help mitigate host tracking based on stable EUI-64 addresses. Often enabled by default on clients but usually not servers.
* **Application Security:** Remains critical regardless of IP version.

<aside class="notes">
A few other points. While the huge address space stops brute-force scanning of the entire internet, attackers aren't stupid. They will target the prefixes actually allocated to organizations (which are public knowledge) or look for addresses leaked in logs, DNS records, or emails. So, obscurity is not a security strategy. Reconnaissance on local networks using NDP or ICMPv6 is also possible if not properly filtered. We mentioned Privacy Addresses earlier – these temporary Interface IDs generated by client OSes help prevent tracking users based on their IPv6 address changing as they move between networks. And of course, application-level security remains just as important as ever, regardless of whether traffic flows over IPv4 or IPv6.
</aside>

---

### Quiz 6: Management & Security

<aside class="notes">
That covers the main management and security topics for Day 3. Let's do our final quiz covering these operational aspects.
(Administer Quiz 6 - ~10 mins)
Any questions before we start Lab 6 on security configuration?
</aside>


---

### Lab 6: IPv6 Security Configuration (NACLs & SGs)

*(Afternoon Lab)*

<aside class="notes">
Now it's time for Lab 6, where you'll apply the firewalling concepts we just discussed. You'll configure AWS Network ACLs and Security Groups to control IPv6 traffic between instances. Please refer to your Lab Guide.
</aside>


---

### Course Wrap-up & Q/A

* **Summary:** Covered IPv6 Fundamentals, Addressing, Protocol, Migration (Dual-Stack, Tunneling, Translation), Management (IPAM, Monitoring, Troubleshooting, DNS/DHCPv6), and Security.
* **Labs:** Explored AWS environment, tested connectivity, configured IPv6 enablement, NLB ingress, dual-stack, tunneling, and security rules.
* **Next Steps:** Continue exploring, test in labs, plan your own organization's transition!

<aside class="notes">
We've reached the end of our three days! We started with the fundamentals of why IPv6 exists and how it works, explored the intricacies of addressing and the protocol itself. We covered the key strategies for migrating and coexisting with IPv4 – Dual-Stack, Tunneling, and Translation. And today, we focused on the practical aspects of managing and securing IPv6 networks. Through the labs, you got hands-on experience exploring an IPv4 environment, manually enabling IPv6, configuring different subnet types (IPv6-only, public/private dual-stack), setting up ingress via NLB, building a tunnel, and implementing firewall rules. Hopefully, this gives you a solid foundation to continue learning and start planning for IPv6 in your own environments. Thank you for your participation! Are there any final questions?
</aside>
