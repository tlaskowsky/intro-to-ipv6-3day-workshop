---
theme: white # Or your preferred theme
revealjs:
  width: 960
  height: 700
  margin: 0.1 # Adjust margin as needed
  center: true
---

## Lab 3 Rationale: Ingress to IPv6-Only

* **Recap Lab 2:** You enabled IPv6 on the `ServiceProviderVPC`, created **IPv6-only private subnets**, and configured **Egress** connectivity (via EIGW & NAT64/NGW). Your instances can reach the internet.
* **The Challenge:** How do external clients (IPv4 or IPv6) connect **IN** to services running on these private, IPv6-only instances?
* **The Solution:** Use a Load Balancer as a secure entry point. Lab 3 focuses on using an AWS Network Load Balancer (NLB) for this **Ingress** traffic.

<aside class="notes">
Let's set the stage for Lab 3. In Lab 2, you successfully configured your IPv6-only instances in the Service Provider VPC to initiate connections *out* to the internet. But what about the reverse? How can users or other services connect *in* to an application hosted on those instances, given they are in private subnets with no direct inbound path? This lab addresses that exact scenario by introducing a Network Load Balancer. We'll configure the NLB to act as the public-facing entry point, securely forwarding incoming requests to our private IPv6-only backend instances. This bridges the gap from configuring egress to enabling ingress.
</aside>

---

## Lab 3 Objectives: NLB Ingress

* Configure an internet-facing **Network Load Balancer (NLB)** with **Dual-Stack** IP addresses.
* Create an **IPv6 Listener** on the NLB to accept incoming traffic (e.g., HTTP).
* Create a **Target Group** that registers your **private IPv6-only instances** (from Lab 2) as targets using their IPv6 addresses.
* Test and verify **inbound connectivity** from the internet to your service via the NLB's public IPv6 address.

<aside class="notes">
Specifically, in this lab, you will perform these key tasks: First, you'll create an NLB and configure it to be dual-stack, meaning it will get both public IPv4 and IPv6 addresses from AWS. Then, you'll set up a listener specifically for IPv6 traffic on a chosen port (like port 80 for HTTP). Next, you'll create a target group and register the IPv6 addresses of the instances running in your IPv6-only private subnets (the ones created in Lab 2) as the targets for this group. Finally, you'll link the listener to the target group and test access by hitting the NLB's public IPv6 address from a browser or using curl, verifying that traffic is correctly forwarded to your backend instances.
</aside>

---

## Lab 3 Reference

* This lab follows the steps outlined in the AWS Workshop module:
    * **Module 2: Ingress connectivity to an IPv6-only instance using NLB**
* Link: [`https://catalog.workshops.aws/ipv6-on-aws/en-US/lab-2`](https://catalog.workshops.aws/ipv6-on-aws/en-US/lab-2)
* *(Please refer to your specific Lab Guide document for detailed step-by-step instructions tailored to our environment).*

<aside class="notes">
The procedures we'll follow are based directly on Module 2 (labeled as Lab 2 in the URL) of the public AWS "IPv6 on AWS" workshop, which focuses on this NLB ingress scenario. The link is provided here for reference. However, please make sure you follow the specific step-by-step instructions provided in *our* workshop's Lab Guide document, as it will contain details specific to the accounts and resources we are using today. Let's begin Lab 3!
</aside>
