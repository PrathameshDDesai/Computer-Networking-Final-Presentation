# Network Slicing in 5G and 6G: Architecture, Protocols, Lifecycle & Interactive Explorer

[![Course](https://img.shields.io/badge/Course-PCCO302%20Computer%20Networks-0284c7.svg)](#academic-credits)
[![Institution](https://img.shields.io/badge/Institution-Sanjivani%20College%20of%20Engineering-darkblue.svg)](#academic-credits)
[![Standards](https://img.shields.io/badge/3GPP-Rel%2015%E2%80%9318%20%7C%20ITU--R%20M.2160-emerald.svg)](#standards--references)
[![Interactive Explorer](https://img.shields.io/badge/Simulator-Interactive%20HTML5%20Canvas-f97316.svg)](./Network%20Slicing%20Explorer.html)
[![License](https://img.shields.io/badge/License-Academic%20Study-gray.svg)](#acknowledgements)

---

## Table of Contents
1. [Executive Summary & Problem Statement](#1-executive-summary--problem-statement)
2. [What is Network Slicing?](#2-what-is-network-slicing)
   - [Definition & Core Intuition](#21-definition--core-intuition)
   - [Network Slicing vs. Traditional QoS / DiffServ / VLANs](#22-network-slicing-vs-traditional-qos--diffserv--vlans)
   - [Slice Identification: S-NSSAI (SST & SD)](#23-slice-identification-s-nssai-sst--sd)
3. [The Three Canonical 5G Slice Profiles](#3-the-three-canonical-5g-slice-profiles)
   - [eMBB: Enhanced Mobile Broadband (SST 1)](#31-embb-enhanced-mobile-broadband-sst-1)
   - [URLLC: Ultra-Reliable Low-Latency Communications (SST 2)](#32-urllc-ultra-reliable-low-latency-communications-sst-2)
   - [mMTC: Massive Machine-Type Communications (SST 3)](#33-mmtc-massive-machine-type-communications-sst-3)
   - [Comparative Slice KPI Profile Matrix](#34-comparative-slice-kpi-profile-matrix)
4. [End-to-End Architectural Framework](#4-end-to-end-architectural-framework)
   - [Radio Access Network (RAN / gNodeB) Slicing](#41-radio-access-network-ran--gnodeb-slicing)
   - [Transport Network Slicing (FlexE, TSN, SRv6)](#42-transport-network-slicing-flexe-tsn-srv6)
   - [5G Core (5GC) Service-Based Architecture (SBA)](#43-5g-core-5gc-service-based-architecture-sba)
   - [Edge Computing (MEC) Integration](#44-edge-computing-mec-integration)
5. [End-to-End Signaling & Slice Selection Walkthrough](#5-end-to-end-signaling--slice-selection-walkthrough)
6. [Resource Allocation, Isolation & Noisy Neighbor Mitigation](#6-resource-allocation-isolation--noisy-neighbor-mitigation)
   - [Hard vs. Soft Isolation](#61-hard-vs-soft-isolation)
   - [Plane-by-Plane Isolation (Control, User, Management)](#62-plane-by-plane-isolation-control-user-management)
   - [Queueing Dynamics & Over-the-Top Bursts](#63-queueing-dynamics--over-the-top-bursts)
7. [Slice Lifecycle Management & Orchestration (MANO)](#7-slice-lifecycle-management--orchestration-mano)
8. [Evolution toward 6G (IMT-2030)](#8-evolution-toward-6g-imt-2030)
   - [Limitations of 5G Slicing](#81-limitations-of-5g-slicing)
   - [Emerging 6G Service Dimensions](#82-emerging-6g-service-dimensions)
   - [AI-Native Zero-Touch Slicing](#83-ai-native-zero-touch-slicing)
   - [Quantitative Performance Leap: 5G vs. 6G Research Targets](#84-quantitative-performance-leap-5g-vs-6g-research-targets)
9. [Interactive Simulation Tool: Network Slicing Explorer](#9-interactive-simulation-tool-network-slicing-explorer)
   - [Overview & Architecture](#91-overview--architecture)
   - [Key Interactive Visualizations & Models](#92-key-interactive-visualizations--models)
   - [How to Launch and Explore](#93-how-to-launch-and-explore)
10. [Repository Structure](#10-repository-structure)
11. [Academic Credits & Project Details](#11-academic-credits--project-details)
12. [Standards & References](#12-standards--references)

---

## 1. Executive Summary & Problem Statement

In legacy mobile networks (2G, 3G, 4G LTE), the network was fundamentally engineered as a **one-size-fits-all, best-effort pipe**. A single monolithic network architecture carried phone calls, background telemetry, web browsing, and streaming video using uniform scheduling policies and centralized core gateways.

Modern mobile communications, however, simultaneously serve drastically divergent and conflicting application classes:
- High-definition video streaming and cloud gaming demand **tens to hundreds of Megabits per second of sustained throughput**, but can tolerate buffering and tens of milliseconds of latency.
- Autonomous vehicles, industrial robotic automation, and remote surgery demand **end-to-end latency below 1 ms with 99.999% reliability**, yet require minimal bandwidth per control packet.
- Massive Internet of Things (IoT) sensors and smart metering demand support for **over 1,000,000 devices per square kilometer**, consuming micro-watts of sleep power and transmitting tiny bursts once every few hours or days.

Building separate physical infrastructures for each service category is commercially unviable and spectral-inefficient. **Network Slicing** solves this dilemma by enabling telecom operators to logically partition a single shared physical network into multiple independent, isolated, and specialized virtual networks ("slices"), each customized end-to-end to satisfy a specific Service Level Agreement (SLA).

```
                            ┌────────────────────────────────────────┐
                            │    PHYSICAL INFRASTRUCTURE (SHARED)    │
                            │  Antennas, Optical Fiber, Data Centers │
                            └───────────────────┬────────────────────┘
                                                │
         ┌──────────────────────────────────────┼──────────────────────────────────────┐
         ▼                                      ▼                                      ▼
┌──────────────────┐                  ┌──────────────────┐                  ┌──────────────────┐
│    eMBB SLICE    │                  │   URLLC SLICE    │                  │    mMTC SLICE    │
│ (Broadband Pipe) │                  │ (Ultra-Low Delay)│                  │(Massive Sensors) │
│ - 100+ Mbps      │                  │ - <1 ms Latency  │                  │ - 10^6 dev/km²   │
│ - 4K/8K Video    │                  │ - 99.999% Up     │                  │ - 10+ Yr Battery │
│ - Central UPF    │                  │ - Edge UPF & MEC │                  │ - Light Core     │
└──────────────────┘                  └──────────────────┘                  └──────────────────┘
```

---

## 2. What is Network Slicing?

### 2.1 Definition & Core Intuition
According to **3GPP TS 23.501** and **TS 28.530**, a **Network Slice** is a complete, logical end-to-end network that provides specific network capabilities and network characteristics. It encompasses:
1. **Dedicated and shared radio resources** at the gNodeB.
2. **Dedicated transport paths** across front-haul, mid-haul, and back-haul links.
3. **Dedicated or shared 5G Core Network Functions (NFs)** (such as AMF, SMF, UPF).
4. **Custom service logic and compute environments** (Cloud / Multi-Access Edge Computing).

### 2.2 Network Slicing vs. Traditional QoS / DiffServ / VLANs
A common misconception in computer networking is that network slicing is simply Quality of Service (QoS) rebranding (such as DiffServ / DSCP bits or IEEE 802.1Q VLANs). In reality, network slicing fundamentally differs across architectural boundaries:

| Dimension | Traditional QoS (DiffServ / IntServ) | Traditional VLAN / VPN | 5G / 6G Network Slicing |
| :--- | :--- | :--- | :--- |
| **Scope** | Hop-by-hop packet prioritization in shared routers | Layer 2/3 packet tagging and tunnel isolation | End-to-end (Air interface + Transport + Core NFs + Cloud) |
| **Control Plane** | Shared control plane; no control plane customization | Shared routing protocol instances | Slice-specific control functions (dedicated SMFs, PCFs) |
| **Compute & Functions** | Cannot instantiate custom network functions | Fixed topology; cannot relocate processing | Dynamically instantiated virtualized/containerized NFs |
| **Lifecycle** | Static configuration on network interfaces | Manually or semi-statically provisioned | Full dynamic MANO lifecycle (instantiate, scale, terminate) |
| **Fault Isolation** | Packet bursts can still starve low-priority queues | Traffic segregated, but shared CPU/buffer exhaustion | Guaranteed compute, buffer, and radio PRB isolation |

### 2.3 Slice Identification: S-NSSAI (SST & SD)
In 3GPP 5G specifications, every slice is globally or regionally identified by **S-NSSAI** (*Single Network Slice Selection Assistance Information*):

$$\text{S-NSSAI} = [\text{SST (8 bits)}] + [\text{SD (24 bits, optional)}]$$

- **SST (Slice/Service Type)**: Standardized 8-bit integer specifying the expected slice behavior:
  - **SST = 1**: eMBB (*enhanced Mobile Broadband*)
  - **SST = 2**: URLLC (*Ultra-Reliable Low-Latency Communications*)
  - **SST = 3**: mMTC (*massive Machine-Type Communications*)
  - **SST = 4**: V2X (*Vehicle-to-Everything communications*)
  - **Values 128–255**: Operator-specific proprietary slice types
- **SD (Slice Differentiator)**: Optional 24-bit hexadecimal value that differentiates between multiple slices of the same SST (e.g., distinguishing an enterprise automotive slice from a public emergency URLLC slice).
- **NSSAI**: A collection of up to 8 S-NSSAIs simultaneously allocated to a single User Equipment (UE) session.

---

## 3. The Three Canonical 5G Slice Profiles

The International Telecommunication Union (ITU-R) in Recommendation **M.2083 (IMT-2020)** formulated three canonical usage scenarios:

```
                                  eMBB
                             (Gigabit Rates)
                                   /\
                                  /  \
                                 /    \
                                /      \
                               /        \
                              /          \
                             /   5G / 6G  \
                            /              \
                           /________________\
                        URLLC              mMTC
                  (Sub-ms, 99.999%)   (10^6 Devices/km²)
```

### 3.1 eMBB: Enhanced Mobile Broadband (SST 1)
- **Primary Goal**: Extremely high data transmission rates and broad spectral efficiency.
- **Key Applications**: 4K/8K video streaming, immersive Virtual Reality (VR), Augmented Reality (AR), cloud workstation streaming.
- **Architectural Characteristics**:
  - Broad carrier channels (100 MHz in Sub-6 GHz, 400–800 MHz in mmWave).
  - Massive MIMO (64T64R) beamforming.
  - User Plane Function (UPF) placed in regional/centralized cloud data centers near Content Delivery Networks (CDNs).
  - Maximizes aggregate throughput over individual packet delay predictability.

### 3.2 URLLC: Ultra-Reliable Low-Latency Communications (SST 2)
- **Primary Goal**: Mission-critical deterministic delivery with stringent delay and loss thresholds.
- **Key Applications**: Tele-surgery, industrial closed-loop robotics, autonomous driving collision avoidance, smart electrical grid fault isolation.
- **Architectural Characteristics**:
  - **Mini-slots and short Transmission Time Intervals (TTI)**: Reducing radio frame scheduling delay from 1 ms to 0.125 ms.
  - **Preemptive Scheduling**: URLLC traffic can puncture ongoing eMBB transmissions in the radio frame.
  - **Edge UPF & MEC**: User Plane Function and application servers are colocated right at the base station or edge site to eliminate backbone fiber transit delay.
  - **Dual Connectivity & Packet Duplication**: Duplicate packet copies transmitted across independent radio paths (PDCP layer duplication) so that loss on one link does not trigger retransmission timeout.

### 3.3 mMTC: Massive Machine-Type Communications (SST 3)
- **Primary Goal**: Ultra-high connection density with minimal energy consumption and low device hardware cost.
- **Key Applications**: Environmental monitoring, smart agriculture soil sensors, smart water and electricity metering, asset tracking.
- **Architectural Characteristics**:
  - **Massive Connection Density**: Engineered to support $10^6 \text{ devices}/\text{km}^2$.
  - **Power Saving Modes (PSM) & eDRX**: Devices stay in deep sleep drawing under $5\ \mu\text{W}$, waking up periodically for short data bursts.
  - **Lightweight Signaling Core**: Control-plane Cellular IoT (CIoT) optimizations that bypass complex user-plane session establishment and transport data directly within NAS signaling messages.
  - **Battery Life**: Exceeding 10 to 15 years on a standard 5 Wh non-rechargeable cell.

### 3.4 Comparative Slice KPI Profile Matrix

| KPI Parameter | eMBB (SST 1) | URLLC (SST 2) | mMTC (SST 3) |
| :--- | :--- | :--- | :--- |
| **Peak Data Rate** | DL: 20 Gbps / UL: 10 Gbps | DL: 100 Mbps / UL: 50 Mbps | DL: ~250 kbps / UL: ~250 kbps |
| **User Experienced Throughput** | 100 Mbps (everywhere) | 10–50 Mbps | 1–100 kbps |
| **Radio Latency (One-way)** | 4 ms | **0.5–1.0 ms** | Non-critical (seconds to minutes) |
| **Reliability** | $10^{-3}$ packet loss | **$99.999\%$ to $99.9999\%$** ($10^{-5}$ to $10^{-6}$ loss) | $10^{-1}$ to $10^{-2}$ |
| **Connection Density** | $10^4 \text{ devices}/\text{km}^2$ | $10^3 \text{ devices}/\text{km}^2$ | **$10^6 \text{ devices}/\text{km}^2$** |
| **Mobility Target** | Up to 500 km/h (High-speed train) | Up to 500 km/h | Stationary to pedestrian |
| **Battery Life Target** | 1–2 days | Days to weeks | **10–15 years** |
| **Core UPF Placement** | Centralized / Regional Data Center | **Base Station Edge / MEC** | Centralized IoT gateway |

---

## 4. End-to-End Architectural Framework

To guarantee strict isolation and SLA fulfillment, network slicing must span all four architectural domains of modern cellular networks:

```
[UE] === (Air Interface) ===> [gNodeB (RAN)] === (Fronthaul/Backhaul) ===> [Transport (SRv6/FlexE)] ===> [5G Core] ===> [DN / Cloud]
                               │                                                 │                          │
                         - PRB Slicing                                    - FlexE TDM              - Dedicated SMF/UPF
                         - Preemption                                     - SRv6 SIDs              - Common AMF / NSSF
```

### 4.1 Radio Access Network (RAN / gNodeB) Slicing
The RAN is the most constrained domain due to limited radio spectrum. RAN slicing employs:
- **Physical Resource Block (PRB) Reservation**: Allocating dedicated subcarriers and time slots to specific slices.
- **Dynamic Prioritization & Puncturing**: Allowing incoming URLLC control packets to immediately preempt (puncture) non-critical eMBB subcarriers.
- **O-RAN Disaggregation**: Partitioning the radio into Central Unit (CU), Distributed Unit (DU), and Radio Unit (RU), allowing slice-specific DU scheduling algorithms.

### 4.2 Transport Network Slicing (FlexE, TSN, SRv6)
Connecting base stations to data centers requires deterministic transit:
- **FlexE (Flexible Ethernet)**: Employs time-division multiplexing (TDM) at the physical layer to create hard-isolated sub-rate channels over physical fiber, preventing buffer collisions.
- **Time-Sensitive Networking (TSN)**: IEEE 802.1Q standards (e.g., 802.1Qbv time-aware shapers) for zero-jitter packet scheduling.
- **Segment Routing over IPv6 (SRv6)**: Attaching explicit segment identifiers (SIDs) to packet headers to steer slice traffic along dedicated, low-latency, geographically shortest optical paths.

### 4.3 5G Core (5GC) Service-Based Architecture (SBA)
The 5GC replaces legacy hardware nodes with virtualized, modular **Network Functions (NFs)** communicating over HTTP/2 RESTful APIs:
- **AMF (Access and Mobility Management Function)**: Single termination point for control plane signaling from the UE; handles registration and routes slice sessions.
- **NSSF (Network Slice Selection Function)**: The brains of slicing; maps UE subscription, requested S-NSSAIs, and operator policies to the target network slice instances.
- **SMF (Session Management Function)**: Instantiated per slice or shared; manages PDU session IP allocation, QoS rules, and tunnels.
- **UPF (User Plane Function)**: The data packet engine; converts encapsulation tunnels to raw IP. Deployed centrally for eMBB and distributed to edge sites for URLLC.
- **NRF (Network Repository Function)**: Service registry allowing NFs to discover slice-specific instances.
- **PCF (Policy Control Function)**: Enforces QoS profiles, flow bitrates, and charging rules per slice.
- **UDM (Unified Data Management)**: Houses subscriber slice subscription profiles.

### 4.4 Edge Computing (MEC) Integration
Multi-Access Edge Computing brings cloud workloads into close physical proximity with the gNodeB. In a URLLC slice:
- Routing packets through the entire national Internet core introduces 20–60 ms of propagation and switching delay.
- Slicing steers URLLC traffic via a local UPF breakout directly into a local MEC server, achieving sub-1 ms round-trip execution for mission-critical automation.

---

## 5. End-to-End Signaling & Slice Selection Walkthrough

The following 6-step sequence illustrates how a user device requests, negotiates, and connects to an isolated slice:

```
[UE]               [gNodeB]              [AMF]               [NSSF]              [SMF]               [UPF]
 │                    │                    │                    │                   │                   │
 │── 1. Registration ─>│                    │                    │                   │                   │
 │   (Requested NSSAI)│                    │                    │                   │                   │
 │                    │── 2. Forward Reg ──>│                    │                   │                   │
 │                    │                    │── 3. Slice Query ─>│                   │                   │
 │                    │                    │   (Subscr + Loc)   │                   │                   │
 │                    │                    │<── 4. Allowed NSSAI─│                   │                   │
 │                    │<── 5. Reg Accept ───│                    │                   │                   │
 │<── Allowed NSSAI ──│                    │                    │                   │                   │
 │                    │                    │                    │                   │                   │
 │── 6. PDU Session Est. (S-NSSAI) ────────>│                    │                   │                   │
 │                    │                    │── Select SMF ─────────────────────────>│                   │
 │                    │                    │                    │                   │── Establish N3 ──>│
 │════════════════ User Plane Data Traffic (Isolated Tunnel) ══════════════════════════════════════════>│
```

1. **Step 1: Device Asks for a Slice (UE $\rightarrow$ gNodeB)**  
   When powering on or opening an app, the UE transmits a `Registration Request` containing the **Requested NSSAI** (the list of S-NSSAIs the device wishes to use).
2. **Step 2: Base Station Picks an AMF (gNodeB $\rightarrow$ AMF)**  
   The gNodeB examines the radio layer slice parameters and forwards the initial NAS message to an appropriate initial AMF that supports the requested slice family.
3. **Step 3: Core Checks Allowed Slices (AMF $\rightarrow$ NSSF)**  
   The AMF queries the **NSSF**, supplying the Requested NSSAI, the subscriber's permanent identity, and current tracking area.
4. **Step 4: NSSF Selects Slice Instances (NSSF $\rightarrow$ AMF)**  
   The NSSF consults the operator's slicing policy and the user's UDM subscription profile. It returns the **Allowed NSSAI** (slices permitted in this geographical area) and target slice instance IDs.
5. **Step 5: Allowed Slices Returned to Device (AMF $\rightarrow$ UE)**  
   The AMF issues a `Registration Accept` message informing the UE of its active Allowed NSSAI.
6. **Step 6: PDU Session Establishment & Traffic Flow (UE $\rightarrow$ SMF $\rightarrow$ UPF)**  
   When the UE initiates data transmission for an application (e.g., an autonomous driving video feed), it specifies the corresponding S-NSSAI (e.g., SST=2). The AMF binds the session to a slice-dedicated SMF, which provisions an isolated data plane tunnel on a dedicated UPF.

---

## 6. Resource Allocation, Isolation & Noisy Neighbor Mitigation

### 6.1 Hard vs. Soft Isolation
- **Hard Isolation**: Dedicated physical hardware, dedicated optical wavelengths, or statically partitioned radio PRBs. Completely eliminates cross-tenant interference but results in lower resource utilization and higher cost.
- **Soft Isolation**: Statistical multiplexing where resources are dynamically pooled across slices using priority queues and minimum guaranteed reservations. Higher spectral efficiency, but requires sophisticated admission control to avoid SLA degradation.

### 6.2 Plane-by-Plane Isolation (Control, User, Management)
- **Control Plane Isolation**: Dedicated signaling queues and separate AMF/SMF instances prevent signaling storms (e.g., millions of IoT sensors waking simultaneously) from exhausting session management resources used by emergency URLLC services.
- **User Plane Isolation**: Dedicated memory buffers, separate routing tables (VRFs), and isolated UPFs ensure that multi-gigabit eMBB file transfers cannot fill network queues and inflate URLLC latency.
- **Management Plane Isolation**: Multi-tenant orchestration portals ensure that tenant slice administrators can view their own metrics without accessing other tenants' data or operational topologies.

### 6.3 Queueing Dynamics & Over-the-Top Bursts
Without network slicing, when a high-bandwidth eMBB video user suddenly initiates a multi-gigabit download, router and base station packet buffers experience **bufferbloat**. Control packets from low-latency URLLC streams get trapped behind large video frames in common FIFO queues, causing latency to spike from <1 ms to over 30–50 ms.

With **active network slicing**, radio resources and buffers are segregated:
- URLLC maintains a reserved resource share (e.g., 20% minimum floor) and separate non-blocking queues.
- An eMBB burst is immediately throttled or shaped within its own slice boundary, maintaining URLLC latency below the 1 ms SLA threshold under all load conditions.

---

## 7. Slice Lifecycle Management & Orchestration (MANO)

3GPP TS 28.530 specifies four formal stages in the network slice lifecycle:

```
┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐
│ 1. PREPARATION  │ ───> │2. COMMISSIONING │ ───> │  3. OPERATION   │ ───> │4. DECOMMISSION  │
│ - Design SLA    │      │ - Instantiate NF│      │ - Monitor KPIs  │      │ - Release PRBs  │
│ - Onboard VNFs  │      │ - Allocate PRBs │      │ - Scale Out/In  │      │ - Reclaim Cloud │
│ - Capacity Plan │      │ - Test & Activate│     │ - Self-Healing  │      │ - Audit Logs    │
└─────────────────┘      └─────────────────┘      └─────────────────┘      └─────────────────┘
```

1. **Preparation**: Defining the network slice template (NST), specifying required bandwidth, latency ceilings, reliability objectives, and geographical boundaries.
2. **Commissioning**: Orchestrator (NFVO / CSMF / NSMF / NSSMF) automatically deploys virtual machines or Kubernetes containers for the NFs, configures transport routing tunnels (SRv6), and reserves RAN PRB quotas.
3. **Operation**: Continuous closed-loop monitoring via **NWDAF** (Network Data Analytics Function). When telemetry indicates slice overload, automated scaling rules dynamically inject additional UPF instances or adjust PRB allocations.
4. **Decommissioning**: Graceful session migration, reclaiming reserved resources, deleting temporary routing paths, and archiving SLA performance records.

---

## 8. Evolution toward 6G (IMT-2030)

### 8.1 Limitations of 5G Slicing
While 5G standardized network slicing, real-world deployments encounter clear physical and architectural constraints:
- Slicing configurations across multi-domain and multi-vendor networks require manual coordination and slow orchestration loops.
- Static slice instantiation lacks fine-grained real-time adaptiveness to rapid channel fluctuations.
- Air interface capabilities hit thermal noise and spectrum limits in Sub-6 GHz and mmWave frequencies.

### 8.2 Emerging 6G Service Dimensions
ITU-R Recommendation **M.2160 (IMT-2030 Framework)** expands the 5G triangle into an expanded hexagon with six key usage scenarios:

```
                      Immersive Communication
                             (eMBB+)
                                /\
                               /  \
     AI & Communication       /    \       Ubiquitous Connectivity
            (New)            /  6G  \            (NTN / Space)
              \             /        \              /
               \           /__________\            /
                \         /            \          /
                 \       /              \        /
            Hyper-Reliable              Massive Communication
               (URLLC+)                        (mMTC+)
                          \            /
                           \          /
                       Integrated Sensing &
                          Communication
                              (ISAC)
```

1. **Immersive Communication**: Holographic teleportation, full-body tactile haptics, multi-sensory virtual environments requiring Tbps aggregate throughput.
2. **Hyper-Reliable Low-Latency Communication (HRLLC)**: Sub-0.1 ms latency with $10^{-7}$ (99.99999%) reliability for coordinated swarm robotics and microsecond grid protection.
3. **Massive Communication**: Connection densities of up to $10^7 \text{ to } 10^8 \text{ devices}/\text{km}^2$.
4. **Ubiquitous Connectivity**: Seamless integration of Non-Terrestrial Networks (NTN)—including Low Earth Orbit (LEO) satellite constellations, high-altitude platform stations (HAPS), and terrestrial cells—into a unified 3D slice.
5. **Integrated Sensing and Communication (ISAC)**: The network acts as a distributed radar sensor, imaging physical environments, detecting road obstacles, and measuring millimeter-scale displacements using the same RF signals used for data.
6. **AI and Communication**: Deep learning models distributed natively across the air interface, enabling federated learning and generative edge inference.

### 8.3 AI-Native Zero-Touch Slicing
In 6G, manual lifecycle orchestration is replaced by **AI-native autonomous control**:
- Machine learning agents embedded within individual gNodeBs predict user mobility and interference patterns seconds in advance.
- Slices are dynamically spawned, adapted, and retired on sub-second timescales without human intervention (*Zero-Touch Network & Service Management - ZSM*).

### 8.4 Quantitative Performance Leap: 5G vs. 6G Research Targets

| Parameter / KPI | 5G Specification (IMT-2020) | 6G Target Vision (IMT-2030) | Multiplier / Improvement |
| :--- | :--- | :--- | :--- |
| **Peak Data Rate** | 20 Gbit/s | **100–1000 Gbit/s (1 Tbps)** | **10× – 50×** |
| **User Experienced Throughput** | 100 Mbit/s | **1 Gbit/s** | **10×** |
| **Over-the-Air Latency** | 1.0 ms | **0.1 ms (100 µs)** | **10× reduction** |
| **Maximum Jitter** | Milliseconds | **Microseconds ($\mu\text{s}$)** | **100× reduction** |
| **Reliability (Frame Loss Rate)** | $10^{-5}$ ($99.999\%$) | **$10^{-7}$ ($99.99999\%$)** | **100× fewer errors** |
| **Connection Density** | $10^6 \text{ devices}/\text{km}^2$ | **$10^7 - 10^8 \text{ devices}/\text{km}^2$** | **10× – 100×** |
| **Area Traffic Capacity** | $10 \text{ Mbit/s}/\text{m}^2$ | **$100 - 1000 \text{ Mbit/s}/\text{m}^2$** | **10× – 100×** |
| **Spectrum Bands** | Sub-6 GHz, mmWave (24–40 GHz) | Sub-THz (100–300 GHz), Visible Light | Massive new bandwidth |
| **Mobility Support** | 500 km/h (Bullet train) | **1000 km/h (Aviation / Hyperloop)** | **2×** |
| **Sensing Precision** | None (separate sensors) | **Centimeter-level positioning & radar** | Native RF sensing |

---

## 9. Interactive Simulation Tool: Network Slicing Explorer

Included in this repository is **[`Network Slicing Explorer.html`](./Network%20Slicing%20Explorer.html)**, a single-file, interactive simulation and pedagogical tool developed to demonstrate every theoretical concept covered in this project.

### 9.1 Overview & Architecture
- **Single-File Zero-Dependency Design**: Built using pure HTML5, modern CSS3 (custom CSS variables, fluid responsive typography, dark/light theme toggle), and native HTML5 2D Canvas + SVG vector graphics.
- **Accessibility & Motion Controls**: Full keyboard navigation, `aria-label` screen reader tags, and automatic detection of `prefers-reduced-motion` with static mathematical fallbacks.
- **Real-Time Physics & Queueing Engines**: Implements mathematical queueing models, geometric packet flow rendering, and state machines running at 60 FPS.

```
┌────────────────────────────────────────────────────────────────────────┐
│                   NETWORK SLICING EXPLORER (HTML5)                     │
│                                                                        │
│ [1. Hero Visualizer]   Mixed packets sorted into eMBB/URLLC/mMTC       │
│ [2. Slice Lanes]       Radio -> Transport -> Core -> Cloud breakdown   │
│ [3. eMBB Lab]          Video quality vs. concurrent streams capacity   │
│ [4. URLLC Lab]         Latency budget (radio+transport+core) & loss    │
│ [5. mMTC Lab]          Device density slider & battery decay model     │
│ [6. 5GC Stepper]       Interactive 6-step registration & S-NSSAI trace │
│ [7. Isolation Lab]     Live queueing model with eMBB burst vs URLLC    │
│ [8. Matcher Tool]      9 applications matched to optimal slice profile │
│ [9. 6G Evolution]      Interactive log-scale performance leap chart    │
└────────────────────────────────────────────────────────────────────────┘
```

### 9.2 Key Interactive Visualizations & Models
1. **Multi-Slice Traffic Hero Visualizer (`#hero-canvas`)**:
   - Illustrates heterogeneous traffic arriving from a common ingress pipe.
   - Visually routes packets into three differentiated pipes: large high-bandwidth payloads for eMBB, fast tiny packets for URLLC, and intermittent pulses for mMTC.
2. **Layer-by-Layer Architectural Lanes (`#layers-svg`)**:
   - Interactive SVG displaying how Radio, Transport, Core, and Cloud layers uniquely configure themselves for each slice profile (e.g., Short slots vs. Wide carriers; Edge UPF vs. Central UPF).
3. **eMBB Capacity & Stream Calculator (`#embb-canvas`)**:
   - Select between HD (8 Mbps), 4K (25 Mbps), 8K (100 Mbps), and VR (150 Mbps).
   - Dynamically calculates and renders the exact number of concurrent video streams that fit into a standard 1 Gbit/s cell capacity.
4. **URLLC 1 ms Latency Budget & Packet Duplication Lab (`#urllc-canvas`)**:
   - Interactive toggles for *Short radio slots*, *Edge server*, and *Duplicate path*.
   - Live breakdown of latency across Radio, Transport, and Core processing against the 1.0 ms strict ceiling.
   - 200-packet grid simulation demonstrating how dual-path transmission reduces packet loss from $p$ to $p^2$.
5. **mMTC Device Density & Battery Longevity Simulator (`#mmtc-canvas`)**:
   - Density slider ($10^2$ to $10^6$ devices/$\text{km}^2$) coupled with reporting intervals (1 min to 24 hrs).
   - Real-time battery life estimator calculating Joule consumption per transmission against a 5 Wh non-rechargeable battery cell with a $5\ \mu\text{W}$ sleep drain.
6. **Interactive 5GC Signaling Walkthrough (`#stepper`)**:
   - Animated nodal graph highlighting message flows between **UE, gNodeB, AMF, NSSF, SMF, UPF, and Data Network**.
   - Step-through capability for eMBB (SST 1), URLLC (SST 2), and mMTC (SST 3), showing slice-specific UPF locations.
7. **Isolation Lab: Queueing Simulation (`#lab-canvas`)**:
   - Live chart graphing URLLC latency over an 8-second moving window.
   - Trigger an **eMBB 3-second traffic burst**:
     - *With Slicing OFF*: Shared buffer collapses, queue delay explodes, and URLLC latency violates the 1 ms target.
     - *With Slicing ON*: URLLC maintains reserved channel capacity, keeping latency below 1 ms despite peak eMBB saturation.
8. **Application-to-Slice Matcher (`#match`)**:
   - Interactive profiling for 9 real-world applications: 8K Video, Cloud Gaming, Stadium VR, Remote Surgery, Factory Robots, Vehicle Safety, Smart Meters, City Air Sensors, and Farm Soil Sensors.
9. **6G Evolution Log-Scale Range Chart (`#sixg`)**:
   - Visual comparison of 8 key performance indicators from 5G specifications to 6G research targets plotted on a logarithmic scale.

### 9.3 How to Launch and Explore
No web servers or build tools are required. Simply open the file in any modern web browser:

```bash
# On Windows (via PowerShell or Command Prompt)
start "Network Slicing Explorer.html"

# On macOS
open "Network Slicing Explorer.html"

# On Linux
xdg-open "Network Slicing Explorer.html"
```

Alternatively, double-click **`Network Slicing Explorer.html`** in your system file explorer.

---

## 10. Repository Structure

```
d:/CN/ (Repository Root)
├── .gitignore                          # Ignores Office lock files, temporary files, and IDE configs
├── README.md                           # Comprehensive documentation and technical concepts guide
├── Network Slicing Explorer.html       # Full interactive simulation and educational web application
├── Network slicing in 5G and 6G.docx   # Academic project report (Sanjivani College of Engineering)
└── Network_Slicing_5G_6G_2.pptx        # Final presentation slide deck
```

---

## 11. Academic Credits & Project Details

This project and report were created as part of the **Continuous Internal Assessment (CIA)** for the **Computer Networks (PCCO302)** course during the **Academic Year 2026–2027**.

### Academic Institution
- **Institution**: Sanjivani Rural Education Society's **Sanjivani College of Engineering** (An Autonomous Institution)
- **Location**: Kopargaon – 423 603, Maharashtra, India
- **Department**: Department of Computer Engineering
- **Class / Semester**: Third Year (TY) / Semester V
- **Course Code & Name**: **PCCO302: Computer Network**

### Project Group (Group ID: 8)

| Sr. No. | Student Name | Roll No. | PRN No. |
| :---: | :--- | :---: | :---: |
| 1 | **Prathamesh Desai** | — | **0124UCSM1043** |
| 2 | **Rohit Gaware** | — | **0124UCSM1013** |
| 3 | **Om Suryawanshi** | — | **0124UCSM1022** |
| 4 | **Krishna Bachkar** | — | **0124UCSM1025** |

### Academic Mentorship
- **Course In-charge & Project Guide**: **Dr. P. N. Kalavadekar**
- **Head of Department**: **Prof. M. A. Jawale**

---

## 12. Standards & References

1. **3GPP TS 23.501**: *System Architecture for the 5G System (5GS); Stage 2 (Release 15–18)*. 3rd Generation Partnership Project.
2. **3GPP TS 28.530**: *Management and orchestration; Concepts, use cases and requirements for network slicing in mobile networks (Release 16)*.
3. **3GPP TS 23.502**: *Procedures for the 5G System (5GS); Stage 2*.
4. **3GPP TS 38.300**: *NR; NR and NG-RAN Overall Description; Stage 2*.
5. **ITU-R Recommendation M.2083-0 (09/2015)**: *IMT Vision - Framework and overall objectives of the future development of IMT for 2020 and beyond*. International Telecommunication Union.
6. **ITU-R Recommendation M.2160-0 (11/2023)**: *Framework and overall objectives of the future development of IMT for 2030 and beyond*. International Telecommunication Union.
7. **IEEE Communications Society (ComSoc)**: *ITU-R IMT-2030 (6G) Backgrounder and Envisioned Capabilities* (2024).
8. **ETSI GS NFV 002**: *Network Functions Virtualisation (NFV); Architectural Framework*. European Telecommunications Standards Institute.
9. **O-RAN Alliance**: *O-RAN Architecture Description 8.0*, Open RAN Technical Workgroup.

---

### Acknowledgements
We express our sincere gratitude to Sanjivani College of Engineering, the Department of Computer Engineering, and our guide Dr. P. N. Kalavadekar for their support in making this study of Next-Generation Network Slicing possible.
