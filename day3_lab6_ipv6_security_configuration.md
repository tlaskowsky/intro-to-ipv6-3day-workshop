

## Lab 6: IPv6 Security Configuration (Security Groups & Network ACLs)

**Goal:** To understand, configure, and test both stateful (Security Group) and stateless (Network ACL) firewall rules for controlling IPv6 traffic in AWS VPC.

**Prerequisites:**

* Completion of Labs 1-4. We will use instances within the `ServiceProviderVPC`.
* Access via Session Manager to:
    * The **Public Dual-Stack instance** launched in Lab 4 (e.g., `SP-Public-DualStack-Demo`). Let's call this **Instance A**.
    * One of the **Private IPv6-only instances** launched in Lab 2 (e.g., `ServiceProviderVPCIPv6EC2InstanceAZA`). Let's call this **Instance B**.
* Basic understanding of Security Groups and Network ACLs from lectures.

**Scenario:** We will test and control ICMPv6 (ping) traffic between Instance A (Public Dual-Stack) and Instance B (Private IPv6-only) using first Security Groups, then Network ACLs.

---

### Part 1: Security Groups (Stateful Firewall)

#### Step 1.1: Identify Instances & IPs

1.  **Locate Instances:** In the EC2 Console, find Instance A (`SP-Public-DualStack-Demo`) and Instance B (`ServiceProviderVPCIPv6EC2InstanceAZA`).
2.  **Note IPv6 Addresses:**
    * Select Instance A, go to Details, find and copy its **IPv6 GUA**. Let's call this `INSTANCE_A_IPv6`.
    * Select Instance B, go to Details, find and copy its **IPv6 GUA**. Let's call this `INSTANCE_B_IPv6`.
3.  **Identify Security Groups:**
    * Note the Security Group ID attached to Instance A (e.g., `Lab4-Public-Demo-SG`). Let's call this `SG-A`.
    * Note the Security Group ID attached to Instance B (e.g., `IPv6EC2InstanceSecurityGroupServiceProviderVPC`). Let's call this `SG-B`.

---

#### Step 1.2: Initial Connectivity Check (ICMPv6)

**(Instructor Note:** Default SGs created by CFN might already allow internal ICMPv6, or might need adjustment first. Verify baseline).

1.  **Ensure ICMPv6 Allowed (If Needed):** Check the Inbound rules for both `SG-A` and `SG-B`. If not already present, add temporary rules allowing `All ICMP - IPv6` from source `::/0` to *both* SGs to establish a baseline.
2.  **Connect:** Use Session Manager to connect to both Instance A and Instance B.
3.  **Test Ping A -> B:** From Instance A's terminal:
    ```bash
    ping6 -c 4 [INSTANCE_B_IPv6]
    ```
    *(This should succeed if SGs allow ICMPv6).*
4.  **Test Ping B -> A:** From Instance B's terminal:
    ```bash
    ping6 -c 4 [INSTANCE_A_IPv6]
    ```
    *(This should also succeed).*

---

#### Step 1.3: Restrict Inbound ICMPv6 on Instance B

1.  **Modify SG-B:** Go to VPC -> Security Groups -> Select `SG-B`.
2.  **Edit Inbound Rules:** Remove the broad "All ICMP - IPv6" rule (if you added it). Ensure there are *no* rules allowing inbound ICMPv6. *(AWS SGs are default deny, so removing the ALLOW rule is sufficient)*.
3.  **Save Rules.**
4.  **Test Ping A -> B:** From Instance A's terminal:
    ```bash
    ping6 -c 4 [INSTANCE_B_IPv6]
    ```
    *This should now **fail** (timeout).*
5.  **Test Ping B -> A:** From Instance B's terminal:
    ```bash
    ping6 -c 4 [INSTANCE_A_IPv6]
    ```
    *This should still **succeed** (assuming SG-A still allows inbound ICMPv6 replies).* Why? Because SGs are stateful! Instance B initiated the connection, so the return traffic (echo reply) is automatically allowed back through SG-B's state table, even though there's no explicit inbound allow rule.

---

#### Step 1.4: Allow Specific Inbound ICMPv6 on Instance B

1.  **Modify SG-B:** Go back to `SG-B` -> Edit inbound rules.
2.  **Add Specific Rule:** Click "Add rule".
    * Type: `All ICMP - IPv6`
    * Protocol: `IPv6-ICMP` (58)
    * Port Range: `All`
    * Source type: `Custom`
    * Source: Enter `[INSTANCE_A_IPv6]/128` (Use the specific IPv6 address of Instance A).
3.  **Save Rules.**
4.  **Test Ping A -> B:** From Instance A's terminal:
    ```bash
    ping6 -c 4 [INSTANCE_B_IPv6]
    ```
    *This should now **succeed**.*
5.  **Test Ping from Another Source (if possible):** If you had a third instance, pinging B from it would still fail, proving only A is allowed.



---

### Part 2: Network ACLs (Stateless Firewall)

**(Instructor Note:** Best practice is often to leave NACLs relatively open (like the default) and rely on SGs for primary filtering, but this demonstrates NACL functionality).

#### Step 2.1: Identify Subnets & NACLs

1.  **Find Subnets:** In the VPC Console -> Subnets, identify the Subnet IDs for Instance A and Instance B.
2.  **Find NACLs:** Go to Network ACLs. Find the NACL currently associated with **Instance B's subnet**. Note its ID. It's likely the Default NACL. Verify its default rules (100/101 Allow All, * Deny All).

---

#### Step 2.2: Create and Associate Custom NACL

1.  **Create NACL:** Click "Create network ACL".
    * Name: `Lab6-Custom-NACL`
    * VPC: Select `ServiceProviderVPC`.
    * Click "Create network ACL".
2.  **Default Rules:** Select the new NACL. Note that its default Inbound and Outbound rules are **DENY ALL**. This is different from the Default NACL for the VPC.
3.  **Associate NACL:** Go to the "Subnet associations" tab for the new NACL. Click "Edit subnet associations". Select the checkbox for **Instance B's subnet**. Click "Save changes".
4.  **Test Ping A -> B:** From Instance A: `ping6 -c 4 [INSTANCE_B_IPv6]`.
    * This should **fail** because the new NACL denies all traffic by default.

---

#### Step 2.3: Allow ICMPv6 Inbound (Stateless Problem)

1.  **Edit Inbound Rules:** Select `Lab6-Custom-NACL`, go to "Inbound rules", click "Edit inbound rules".
2.  **Add Allow Rule:** Click "Add rule".
    * Rule number: `100` (Lower numbers evaluated first).
    * Type: `All ICMP - IPv6`
    * Protocol: `ICMPv6` (58)
    * Port range: `ALL`
    * Source: `::/0` (Allow from anywhere for now)
    * Allow/Deny: `ALLOW`
3.  **Save changes.**
4.  **Test Ping A -> B:** From Instance A: `ping6 -c 4 [INSTANCE_B_IPv6]`.
    * This will likely **still fail** (timeout). Why?

---

#### Step 2.4: Allow ICMPv6 Outbound Reply

1.  **Edit Outbound Rules:** Select `Lab6-Custom-NACL`, go to "Outbound rules", click "Edit outbound rules".
2.  **Add Allow Rule:** Click "Add rule".
    * Rule number: `100`
    * Type: `All ICMP - IPv6`
    * Protocol: `ICMPv6` (58)
    * Port range: `ALL`
    * Destination: `::/0` (Allow reply to anywhere for now)
    * Allow/Deny: `ALLOW`
3.  **Save changes.**
4.  **Test Ping A -> B:** From Instance A: `ping6 -c 4 [INSTANCE_B_IPv6]`.
    * This should **now succeed**.

---

#### Step 2.5: Refine NACL Rules (Optional)

* You could make the NACL rules more specific:
    * **Inbound:** Allow ICMPv6 Type 128 (Echo Request) Source `[INSTANCE_A_IPv6]/128`.
    * **Outbound:** Allow ICMPv6 Type 129 (Echo Reply) Destination `[INSTANCE_A_IPv6]/128`.
* This provides tighter control but requires managing more rules.

---

#### Step 2.6: Cleanup

1.  **Edit Subnet Association:** Go back to the `Lab6-Custom-NACL`, "Subnet associations" tab. Click "Edit subnet associations". **Deselect** Instance B's subnet. Click "Save changes". *(This re-associates the subnet with the Default NACL).*
2.  **Delete Custom NACL:** Select `Lab6-Custom-NACL` from the Network ACLs list and choose Actions -> Delete network ACL. Confirm deletion.
3.  **Cleanup SGs:** Revert any temporary changes made to `SG-A` or `SG-B` in Part 1 if desired.

---

### Lab 6 Summary

* **Security Groups:** Stateful, applied at instance level, default deny, allow return traffic automatically. Good for fine-grained instance protection.
* **Network ACLs:** Stateless, applied at subnet level, default allow (for VPC default NACL) OR default deny (for custom NACLs), require explicit rules for *both* inbound and outbound traffic flows. Good for broad subnet-level blocking/allowing.
* Both support distinct rules for IPv4 and IPv6 traffic.
