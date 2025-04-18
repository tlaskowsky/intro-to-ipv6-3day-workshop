---
theme: white # Or your preferred theme
revealjs:
  width: 960
  height: 700
  margin: 0.1 # Adjust margin as needed
  center: true
---

# Day 1 Lab 2

*Enabling IPv6 & Egress Connectivity*

<aside class="notes">
Welcome to Lab 2. In Lab 1, we explored the initial, IPv4-only configuration of the 'Service Provider VPC'. Now, we're going to start adding IPv6 capabilities to that same VPC, simulating a common real-world scenario.
</aside>

---

## Why This Lab?

- **Lab 1:** Explored the initial **IPv4-only** state of the Service Provider VPC.
Identified VPC CIDR, IPv4 subnets, IGW, NGWs, IPv4 routing.
Verified basic IPv4 connectivity between instances.

- **Lab 2 Goal:** Manually **enable and configure IPv6** on the Service Provider VPC.
Experience the process of adding IPv6 features.
First step: Enable basic IPv6 and configure **egress connectivity** for new **IPv6-only** resources.

<aside class="notes">
So, why are we doing this specific lab now? Lab 1 gave us the "before" picture – a standard IPv4 VPC setup for the Service Provider VPC. Lab 2 is the first part of the "after" picture. Instead of just looking at a pre-configured dual-stack VPC like Shared Services, here you'll perform the actual steps required to introduce IPv6 into an existing IPv4 environment. We'll start by enabling IPv6 at the VPC level and then focus on a common requirement: allowing new, IPv6-only instances (which don't have IPv4 addresses) to initiate connections out to the internet (both IPv6 and IPv4 destinations). This is known as egress connectivity.
</aside>

---

## Lab Objectives

- Understand the steps to **enable IPv6** on an existing AWS VPC.
- **Configure** new **IPv6-only** subnets within the VPC.
- Implement **routing** for IPv6-only subnets:
-- Default IPv6 route (::/0) via an **Egress-Only Internet Gateway (EIGW)**.
-- NAT64 route (64:ff9b::/96) via a **NAT Gateway (NGW)**.
- **Launch** EC2 instances within the IPv6-only subnets.
- **Test and verify** outbound connectivity (egress) from these instances to both IPv6 and IPv4 internet destinations.

<aside class="notes">
Specifically, by the end of this lab, you should understand how to add an IPv6 CIDR block to a VPC that previously only had IPv4. You will then create subnets designed exclusively for IPv6. A key part is setting up the correct routing for these subnets: we need a path for native IPv6 traffic to get out (using an Egress-Only Internet Gateway for security) and a path for traffic destined for the IPv4 internet to get translated (using the NAT64 prefix routed via a NAT Gateway). Finally, you'll launch instances into these new subnets and test that they can indeed reach external resources as expected, verifying both the EIGW and NAT64/NGW paths are working.
</aside>

---

## Lab Reference

- This lab follows the steps outlined in the AWS Workshop module:
-- **Module 1: Egress connectivity from an IPv6-only instance**
- Link: https://catalog.workshops.aws/ipv6-on-aws/en-US/lab-1
