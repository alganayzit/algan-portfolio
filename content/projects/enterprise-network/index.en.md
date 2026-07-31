---
title: "Enterprise Network Design & Simulation"
date: 2026-01-18
description: "A comprehensive campus network architecture showcasing high availability, dynamic routing, and security boundaries."
showDate: true
showAuthor: false
showReadingTime: true
---

![Enterprise Network Topology](featured.png)

### Project Overview
This project focuses on simulating a multi-tier enterprise network architecture designed to meet corporate standards for scalability, redundancy, and structural security. It models a modern campus network with distinct core, distribution, and access layers, ensuring that logical boundaries align with engineering best practices.

### Core Architecture & Technical Approach

#### 1. High Availability & Redundancy (HSRP)
To prevent single points of failure at the default gateway level, **Hot Standby Router Protocol (HSRP)** is implemented at the distribution layer. 
* *Objective:* It provides automated default gateway redundancy for critical VLANs, ensuring uninterrupted traffic flow if a primary layer-3 switch or router goes offline.

#### 2. Dynamic Routing & Scalability (OSPF)
Network-wide reachability is established using Single-Area **OSPF (Open Shortest Path First)**. 
* *Objective:* This approach ensures fast convergence, efficient path selection, and simplifies network expansion. OSPF areas and link costs are tuned to optimize bandwidth utilization across backbone links.

#### 3. Segmentation & Security (VLANs & ACLs)
Broadcast domains are strictly confined using local **VLANs** mapped to specific corporate functions (e.g., Management, IT, Operations, Guest).
* *Objective:* Inter-VLAN routing is strictly controlled via Access Control Lists (ACLs) to enforce a least-privilege access model, filtering unauthorized traffic before it reaches sensitive network segments.

#### 4. Border Security & Internet Edge (NAT)
To facilitate secure internet egress while conserving public IP space, **Network Address Translation (NAT)** — specifically Port Address Translation (PAT) — is configured at the enterprise edge.
* *Objective:* It hides internal IP schemas from the public internet, translating private addresses into reusable public endpoints, thereby establishing a basic layer of boundary defense.

#### 5. Enterprise Mobility & Guest Access (Wi-Fi)
Wireless connectivity is integrated into the campus network design, separating corporate internal traffic from guest traffic.
* *Objective:* Secure authentication mechanisms are simulated to ensure that wireless endpoints are dynamically funneled into their respective VLANs, preventing unauthorized access to core resources.
---

### Technical Stack
<div class="flex flex-wrap gap-2 my-4">
  <span class="bg-slate-100 dark:bg-slate-800 text-slate-800 dark:text-slate-200 text-xs font-medium px-2.5 py-1 rounded border border-slate-200 dark:border-slate-700">Cisco IOS</span>
  <span class="bg-slate-100 dark:bg-slate-800 text-slate-800 dark:text-slate-200 text-xs font-medium px-2.5 py-1 rounded border border-slate-200 dark:border-slate-700">OSPF v2</span>
  <span class="bg-slate-100 dark:bg-slate-800 text-slate-800 dark:text-slate-200 text-xs font-medium px-2.5 py-1 rounded border border-slate-200 dark:border-slate-700">HSRP</span>
  <span class="bg-slate-100 dark:bg-slate-800 text-slate-800 dark:text-slate-200 text-xs font-medium px-2.5 py-1 rounded border border-slate-200 dark:border-slate-700">VLAN Trunking (802.1Q)</span>
  <span class="bg-slate-100 dark:bg-slate-800 text-slate-800 dark:text-slate-200 text-xs font-medium px-2.5 py-1 rounded border border-slate-200 dark:border-slate-700">Extended ACLs</span>
  <span class="bg-slate-100 dark:bg-slate-800 text-slate-800 dark:text-slate-200 text-xs font-medium px-2.5 py-1 rounded border border-slate-200 dark:border-slate-700">Dynamic NAT / PAT</span>
  <span class="bg-slate-100 dark:bg-slate-800 text-slate-800 dark:text-slate-200 text-xs font-medium px-2.5 py-1 rounded border border-slate-200 dark:border-slate-700">Wireless LAN (WLAN)</span>
</div>