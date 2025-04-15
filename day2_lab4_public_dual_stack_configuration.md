---
theme: white # Or your preferred theme
revealjs:
  width: 960
  height: 700
  margin: 0.1 # Adjust margin as needed
  center: true
---

# Lab 4: Public Dual-Stack Configuration

*(Challenge Lab)*

<aside class="notes">
Welcome to our final lab for Day 2 - Lab 4. This lab will be a bit different, presented as a 'challenge'. We'll be configuring the public dual-stack scenario within the Service Provider VPC you've been working on.
</aside>

---

## Lab 4 Rationale: Completing the Picture

* **Revised Recap:**
    * Lab 1: Explored initial **IPv4-only** SP VPC (Private Subnets); tested connectivity between instances in **Private Subnets**.
    * Lab 2: Configured **IPv6-only Private** Subnets + Egress (EIGW/NAT64) in SP VPC.
    * Lab 3: Configured **Ingress** to IPv6-only instances via NLB in SP VPC.
* **Lab 4 Goal:** Configure and test the **Public Dual-Stack** scenario in the SP VPC, where instances have direct IPv4/IPv6 internet access via the **Internet Gateway (IGW)**. This contrasts with the private subnet routing using NGW/EIGW.

<aside class="notes">
Let's connect this lab to our journey so far. In Lab 1, you explored the basic IPv4 setup of the Service Provider VPC, including its public and private subnets, and tested connectivity between the instances located in the *private* subnets. In Lab 2, you enabled IPv6 and specifically configured IPv6-only *private* subnets with secure egress using EIGW and NAT64. In Lab 3, you provided *ingress* to those IPv6-only instances using an NLB. Now, in Lab 4, we'll configure the *public* dual-stack scenario. This involves ensuring instances in public subnets can communicate directly with the internet using both IPv4 and IPv6 via the main Internet Gateway. This completes our look at the primary subnet configurations: IPv4-only private, IPv6-only private, and now public dual-stack.
</aside>

---

## Lab 4 Objectives: Public Dual-Stack via IGW

* **Configure** a **Public Subnet** in the `ServiceProviderVPC` for Dual-Stack operation. *(Modify one of the existing public subnets, ensuring it was made dual-stack in Lab 2)*.
* Ensure the subnet's associated **Route Table** (`ServiceProviderVPCPublicRouteTable`) correctly routes traffic via the **Internet Gateway (IGW)** for *both*:
    * IPv4 Default (`0.0.0.0/0`)
    * IPv6 Default (`::/0`)
* **Launch** an EC2 instance into this public dual-stack subnet.
* Ensure the instance receives both a **Public IPv4 address** and an **IPv6 address**.
* **Test** direct inbound and outbound connectivity to/from the instance over both IPv4 and IPv6.

<aside class="notes">
Your mission in this lab is to set up and verify a public dual-stack configuration. This means taking one of the public subnets in the Service Provider VPC (which you modified to be dual-stack in Lab 2) and ensuring its route table points *both* the IPv4 and IPv6 default routes directly to the Internet Gateway. Then, you'll launch an instance, making sure it gets both a public IPv4 address and an IPv6 address (check subnet settings and instance launch details). Finally, you'll test connectivity – can you SSH to the public IPv4? Can you SSH to the IPv6 address (from an IPv6-capable source)? Can the instance ping/curl external IPv4 and IPv6 destinations directly?
</aside>

---

## Challenge Lab Format!

* This lab provides **objectives and high-level guidance**.
* It includes **fewer detailed step-by-step instructions** compared to previous labs.
* **Goal:** Apply the knowledge and skills gained from Day 1 lectures, Day 2 lectures, and previous labs (Labs 1, 2, 3).
* **Resources:** Refer to your notes, previous lab steps, and AWS documentation if needed. Ask the instructor for hints if you get stuck!

<aside class="notes">
Now, about the 'challenge' format. Unlike the previous labs that likely had very detailed, click-by-click instructions, this lab will primarily give you the objectives (what needs to be achieved) and some pointers or hints. You'll need to figure out the specific steps in the AWS console or CLI by applying what you've learned about VPCs, subnets, route tables, IGWs, EC2 instance settings, and security groups for both IPv4 and IPv6. Think back to how you configured routing or modified subnets in earlier labs. Don't hesitate to consult the AWS documentation or ask for a hint if you get stuck, but the goal is for you to synthesize the steps yourself. Good luck!
</aside>

---

## Lab 4 Reference

* Refer to your specific **Lab Guide document** for the objectives, hints, and expected outcomes.
* Consult relevant AWS Documentation if needed:
    * VPC User Guide (Subnets, Route Tables, IGW)
    * EC2 User Guide (Launching Instances, IP Addressing)

<aside class="notes">
Your primary resource for this lab is the Lab 4 section in your workshop Lab Guide document, which will outline the specific objectives and any hints provided. If you need more detailed information on specific AWS services, the official AWS documentation is always the best source. Let's get started with Lab 4!
</aside>

---

## Optional Lab 4 Extension: Private Dual-Stack

* **Goal:** Configure and test a **private** dual-stack subnet within the `ServiceProviderVPC`.
* **High-Level Steps:**
    1. Create a **new private subnet** (choose non-overlapping IPv4 CIDR, assign IPv6 /64).
    2. Create a **new Route Table**.
    3. Add routes: `0.0.0.0/0` -> **NGW** and `::/0` -> **EIGW**.
    4. Associate Route Table with the new subnet.
    5. Launch EC2 instance (ensure IPv4/IPv6 assigned).
    6. Test outbound connectivity (v4 & v6). Compare accessibility vs. public instance.

<aside class="notes">
If you finish the main objectives for Lab 4 early or want an additional challenge, try this optional extension. The goal here is to create and test a *private* dual-stack subnet, contrasting it with the public one you just configured. You'll need to create a new subnet, making sure to give it both an IPv4 and IPv6 CIDR block. Then, create a dedicated route table for it. The key difference is the routing: point the IPv4 default route to one of the NAT Gateways and the IPv6 default route to the Egress-Only Internet Gateway. Associate this table with your new subnet, launch an instance making sure it gets both IP addresses (it won't get a public IPv4 this time), and test its outbound connectivity. Compare how it reaches the internet versus the instance in the public subnet.
</aside>
d