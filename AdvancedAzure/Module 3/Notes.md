### **Slide 1: Azure Virtual Networks (VNet)**

#### Speaker Notes:

**Opening Talking Points:**

* "Let’s begin our networking deep dive by understanding the foundation: the Azure Virtual Network, or VNet."
* "Think of a VNet as your own private datacenter, but in the cloud."

**Key Concepts:**

* "VNets are logically isolated—meaning they’re your slice of the Azure fabric, and other customers can’t access it."
* "Just like in on-prem networking, you define IP address ranges, subnets, and control traffic flow."

**Real-World Context:**

* "In practice, VNets allow your VMs, containers, databases, and even PaaS resources to securely talk to each other."
* "They’re also the foundation for more complex scenarios—like hybrid connectivity, hub-and-spoke models, and microsegmentation."

**Instructor Cues:**

* Ask: *“Who here has configured a VNet before? What kind of workloads were you running?”*
* If time permits: Draw a quick analogy to VLANs for learners with traditional networking backgrounds.

---

### **Slide 2: VNet Diagrams – Logical View**

#### Speaker Notes:

**Talking Points:**

* "This visual shows a typical layout: a single VNet segmented into multiple subnets, each containing different workloads."
* "Everything here lives inside your subscription, deployed to a specific region."

**Visual Guidance:**

* "The outer red line shows the subscription boundary. Inside that, green indicates the region, and the blue box is your VNet."
* "Each subnet is a smaller slice of IP space that hosts resources like VMs or application gateways."

**Real-World Context:**

* "This setup lets you isolate front-end, back-end, and database layers using subnets—even while they all share a VNet."
* "Each subnet can also have its own NSG and route table for precise traffic control."

**Instructor Cues:**

* Ask: *“Why might we separate workloads into different subnets rather than using a flat IP space?”*
* Tie to security and control: *“Subnet separation enables layered defense and easier network policy enforcement.”*

---

### **Slide 3: VNet Diagrams – Hybrid Scenario**

#### Speaker Notes:

**Talking Points:**

* "Now we’re looking at a more complete hybrid networking scenario. Here we’ve extended our on-prem network into Azure using ExpressRoute."
* "Notice the logical separation between internal (INT) and DMZ subnets."

**Key Concepts:**

* "Subnet 1 hosts domain controllers behind an internal load balancer—this setup is common in hybrid identity models."
* "Subnet 2 is more exposed—it routes through an internet-facing load balancer, protected by a dedicated NSG."

**Real-World Context:**

* "You might see this in regulated industries—finance, healthcare—where only some workloads can face the internet."
* "ExpressRoute or VPN Gateway connects the on-prem data center to Azure with secure, low-latency links."

**Instructor Cues:**

* Ask: *“Can anyone share if your organization uses ExpressRoute or site-to-site VPNs today?”*
* Emphasize the importance of NSGs and subnet-level design in secure architectures.

---

### **Slide 4: VNet Capabilities – Isolation, Customizability, DNS**

#### Speaker Notes:

**Talking Points:**

* "Let’s walk through the core capabilities that make VNets so flexible: isolation, customizability, and name resolution."

**Break It Down:**

* "Each VNet is fully isolated—even from other VNets in your own subscription—unless you explicitly connect them via peering or gateways."
* "You define CIDR blocks, segment with subnets, and design your own address space."

**Real-World Context:**

* "Enterprises often deploy separate VNets for Dev, Test, and Prod—reusing IP ranges across subscriptions or regions."
* "This flexibility supports multi-team environments without conflict."

**Name Resolution Tip:**

* "By default, Azure handles DNS for internal resolution—but you can override this to use custom DNS or private zones."

**Instructor Cues:**

* Ask: *“Has anyone run into DNS issues when connecting hybrid or containerized workloads in Azure?”*
* Optional: Show Azure DNS settings for a VNet if demo time allows.

---

### **Slide 5: VNet Capabilities – Peering, Routing, NSGs**

#### Speaker Notes:

**Talking Points:**

* "Beyond basic subnet communication, VNets can connect with each other using *VNet Peering*—either within a region or globally."
* "This enables you to build larger architectures across multiple VNets without routing traffic through the public internet."

**Key Features Highlight:**

* "Network Security Groups help control which traffic is allowed or denied—kind of like a stateless firewall at the subnet or NIC level."
* "Internal routing allows resources in different subnets to talk to each other without any special setup—unless you override it."

**Real-World Context:**

* "A common use case: app tier in one subnet, database in another—controlled with NSGs and route tables for extra security."
* "Peering also supports hub-and-spoke models—great for centralizing shared services like firewalls or logging."

**Instructor Cues:**

* Ask: *“Can anyone explain the difference between peering and using a VPN Gateway to connect VNets?”*
* Optional: Whiteboard a hub-and-spoke diagram or ask someone to sketch it on the board.

---

### **Slide 6: VNet Diagrams – Gateway-based Connectivity**

#### Speaker Notes:

**Talking Points:**

* "This final diagram shows gateway-based connectivity between two VNets—specifically using VPN Gateway."
* "Note the difference in deployment models—ASM on the left, ARM on the right. Most new deployments use ARM."

**Deployment Nuances:**

* "Each VNet has a gateway subnet and an attached VPN Gateway. These gateways form a secure tunnel between networks."
* "This is useful when peering isn’t viable—for example, cross-subscription scenarios, or connecting to third-party networks."

**Real-World Context:**

* "You’ll often see VPN Gateway used in early-stage migrations when on-prem and cloud need secure connectivity during cutover."
* "Later, teams may switch to ExpressRoute or peering for better performance."

**Instructor Cues:**

* Ask: *“How many of you are still running workloads under ASM, or connecting to legacy networks?”*
* Optional: Share Azure’s own recommendation to migrate from ASM to ARM-based networking.

---

### **Slide 7: VNet Reserved IPs**

#### Speaker Notes:

**Talking Points:**

* "Let’s talk about address space efficiency. Even though a subnet may have 8 or 16 IPs on paper, not all of them are usable."
* "Azure reserves *five* IP addresses per subnet—three for internal platform services, one for the network ID, and one for broadcast."

**Real-World Context:**

* "In this /29 subnet, we technically have 8 addresses—but only 3 are available for VMs or services."
* "This catches a lot of people off guard, especially in small subnets. Planning CIDR blocks is critical."

**Instructor Cues:**

* Ask: *“Anyone ever deployed a small subnet and then wondered why half the VMs failed?”*
* Pro Tip: Encourage students to size subnets with growth in mind—Azure reserves won’t change.

---

### **Slide 8: VNet Protocol Support and Limitations**

#### Speaker Notes:

**Talking Points:**

* "Azure VNets support most common protocols—TCP, UDP, ICMP, ESP, and so on."
* "However, some network-layer traffic types are intentionally blocked."

**Key Restrictions:**

* "Multicast, broadcast, and GRE packets are *not supported*. This limits certain legacy workloads like clustering or older routing protocols."
* "You also can’t use DHCP inside Azure. IPs are statically assigned by Azure’s internal system."

**Real-World Context:**

* "If you’re migrating on-prem systems that depend on these protocols, you’ll need to re-architect or use different tools in Azure."

**Instructor Cues:**

* Ask: *“Has anyone run into blocked protocol issues—especially with legacy apps or vendor appliances?”*
* Optional: Highlight that unicast is fully supported—multicast isn’t needed in most modern designs.

---

### **Slide 9: Traffic Flow in a VNet (Overview)**

#### Speaker Notes:

**Talking Points:**

* "Let’s shift focus to how traffic actually flows within and between VNets."
* "Within a VNet, all resources can talk to each other by default—no extra config needed."

**Key Concepts:**

* "You can connect to other VNets via peering, and to on-prem environments via VPN or ExpressRoute."
* "By default, VMs also have outbound access to the internet—unless you restrict it."

**Real-World Context:**

* "This default behavior is helpful for quick deployments—but in production, you'll want more control."

**Instructor Cues:**

* Ask: *“How many of you are currently using NAT Gateway or Azure Firewall to manage egress traffic?”*
* Optional: Circle back to subnet boundaries and NSGs as tools to enforce traffic flow.

---

### **Slide 10: Traffic Flow in a VNet (Control + Security)**

#### Speaker Notes:

**Talking Points:**

* "Just because everything *can* talk doesn’t mean it *should*. Let’s look at how to enforce traffic boundaries."
* "Multi-tier applications should only allow traffic between adjacent tiers—web to app, app to DB—not full mesh."

**Best Practices:**

* "Use NSGs to enforce port-level rules."
* "Use route tables or firewalls to direct traffic to a specific inspection point, especially for internet egress."

**Real-World Context:**

* "A common mistake: leaving all subnets fully open and then wondering why a lateral movement attack wasn’t contained."

**Instructor Cues:**

* Ask: *“What’s your default policy for east-west traffic—allow all, or deny all?”*
* Optional: Mention Azure Policy to enforce subnet NSG presence or deny open ports.

---

### **Slide 11: Network Security Groups (NSGs)**

#### Speaker Notes:

**Talking Points:**

* "NSGs are your go-to method for traffic filtering in Azure."
* "They work like lightweight firewalls—rule containers that allow or deny traffic based on defined criteria."

**How They're Applied:**

* "NSGs can be assigned to subnets or directly to NICs—or both."
* "Rule enforcement always happens at the NIC level—even if the rule is defined at the subnet."

**Rule Processing Order:**

* "Inbound: Subnet NSG first, then NIC NSG."
* "Outbound: NIC NSG first, then Subnet NSG."
* "Both must allow the traffic, or it’s denied."

**Real-World Context:**

* "A lot of weird issues in Azure networking come down to conflicting NSG rules at subnet vs NIC. Always check both."

**Instructor Cues:**

* Ask: *“Who here prefers subnet-level rules vs NIC-level rules? Why?”*
* Optional: Draw a scenario where one allows and the other denies—see if students can troubleshoot it.

---

### **Slide 12: NSG Rules – The 5-Tuple**

#### Speaker Notes:

**Talking Points:**

* "NSG rules follow the classic 5-tuple matching model—if you’ve worked with firewalls before, this should look familiar."

**Rule Components:**

1. **Source** – IP, CIDR, Service Tag (like Internet or VirtualNetwork), or an ASG.
2. **Source Port** – Typically \*, but can be scoped to specific ranges.
3. **Destination** – IP, CIDR, Service Tag, or ASG.
4. **Destination Port** – Specific port or port range.
5. **Protocol** – TCP, UDP, ICMP, ESP, AH, or \*.

**Real-World Context:**

* "Service Tags are great for abstracting known ranges—like 'AzureCloud' or 'Storage'."
* "Application Security Groups (ASGs) let you target rules to groups of VMs without hardcoding IPs."

**Instructor Cues:**

* Ask: *“How many of you are using ASGs in production right now?”*
* If unfamiliar: Explain ASGs as Azure’s way to group workloads logically for rule targeting.

---

### **Slide 13: NSG Rule Scope with CIDR & Tags**

#### Speaker Notes:

**Talking Points:**

* "Let’s expand on how we define NSG rules. Azure lets us be flexible by targeting IPs directly, or by using broader constructs like CIDR blocks or service tags."

**Scope Options:**

* "Use IPs for precision, CIDRs for ranges, and service tags for abstraction."
* "Tags are predefined by Azure—for example, `VirtualNetwork` means 'anything inside this VNet'."

**Tag Context:**

* "`AzureLoadBalancer` is used to allow health probe traffic from Azure’s internal balancer."
* "`Internet` covers anything outside your VNet—use this with care."

**Instructor Cues:**

* Ask: *“Who here is already using service tags in your NSGs?”*
* Optional demo: Show rule creation in the portal with dropdown for service tag selection.

---

### **Slide 14: NSG Example (Visual Flow)**

#### Speaker Notes:

**Talking Points:**

* "This diagram brings it all together—a classic 3-tier setup using NSGs to control flow."

**Walkthrough:**

* "Internet traffic is only allowed into the frontend subnet."
* "Each tier has its own NSG allowing only the specific upstream/downstream traffic."
* "Back-tier systems can’t be reached directly from the internet."

**Real-World Context:**

* "This layered model mimics what we’d see in secure on-prem designs—segmented workloads, locked-down flows."

**Instructor Cues:**

* Ask: *“Where would you place logging or inspection tools in this architecture?”*
* Optional: Have students sketch similar diagrams using their org’s architecture.

---

### **Slide 15: NSG Example (Rule Table)**

#### Speaker Notes:

**Talking Points:**

* "NSG rules are processed by priority—lower numbers win."
* "Here we see three rules: one to allow from the load balancer, one for RDP from inside the VNet, and a blanket deny."

**Rule Logic:**

* "The final deny rule is a best practice—it ensures no unexpected traffic slips through."
* "Think of it like a catch-all at the end of a firewall rule list."

**Instructor Cues:**

* Ask: *“What happens if two rules match but one has a higher priority number?”*
* Optional: Discuss default rules in Azure that are created automatically and how to override them.

---

### **Slide 16: Application Security Groups (ASGs)**

#### Speaker Notes:

**Talking Points:**

* "Application Security Groups, or ASGs, help us scale NSGs as environments grow."
* "Instead of writing IP-based rules, we group VMs by logical role—like 'WebServers' or 'DBServers'."

**Benefits:**

* "This decouples your security model from the IP layout—great for dynamic or autoscaled workloads."
* "You assign ASGs to NICs, and reference them in NSG rules like you would a tag or IP."

**Real-World Context:**

* "In microservice architectures, roles may shift or scale frequently—ASGs help maintain control without rewriting rules."

**Instructor Cues:**

* Ask: *“Have you ever had to update an NSG because a server changed IP or subnet?”*
* Optional: Demo how to assign a NIC to an ASG in the Azure Portal or via Bicep.

---

### **Slide 17: ASG Example (Visual)**

#### Speaker Notes:

**Talking Points:**

* "This diagram shows a real-world application of ASGs. Each VM is assigned to a group by role."
* "The NSG uses logical group names to define traffic flow rules—like allowing Web to App on HTTPS, or App to DB on port 1443."

**Key Concepts:**

* "Notice that the NSG doesn’t care about IPs or subnets—it’s all group-based."
* "A final 'deny all inbound' rule ensures nothing outside the defined flows is allowed."

**Instructor Cues:**

* Ask: *“What happens if a new VM is spun up with no ASG assigned—will the NSG protect it?”*
* Optional: Discuss automation strategies for ASG/NIC assignment in deployment pipelines.

---

### **Slide 18: Pop Quiz (Scenario Only)**

#### Speaker Notes:

**Reading Prompt:**

* "Let’s put your knowledge into practice with a quick scenario."
* "We’ve got two peered VNets, and we only want a subset of traffic allowed—from DB servers in WestEurope to App servers in EastUS."

**Question Framing:**

* "Your goal: find the *simplest and most secure* way to enforce this requirement."

**Instructor Cues:**

* Ask: *“Take 30 seconds—what's your instinct? Which option stands out?”*
* Then flip to Slide 19 for the answer reveal.

---

### **Slide 19: Pop Quiz (Answer Highlighted)**

#### Speaker Notes:

**Answer Review:**

* "Correct answer: **B**—Block traffic to remote VNet on the peering *and* scope NSG rules to allow only the DB subnet."

**Why This Works:**

* "This avoids overengineering—it’s secure, policy-based, and doesn’t require extra infrastructure like Azure Firewall."
* "Always check what peering settings and NSGs can accomplish before introducing more complexity."

**Instructor Cues:**

* Ask: *“Did anyone choose A or D? Why?” → Use this to highlight common misconceptions about complexity vs simplicity in cloud design.*
