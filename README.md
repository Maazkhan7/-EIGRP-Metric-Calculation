<div align="center">

# 🧮 EIGRP Metric Calculation

### Bandwidth · Delay · Composite Metric · Proving the Router's Number by Hand — Cisco EIGRP AS 1

![Cisco](https://img.shields.io/badge/Cisco-Packet%20Tracer-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![Protocol](https://img.shields.io/badge/Protocol-EIGRP-blue?style=for-the-badge)
![Metric](https://img.shields.io/badge/Metric-2172416-brightgreen?style=for-the-badge)
![AD](https://img.shields.io/badge/AD-90-orange?style=for-the-badge)
![Level](https://img.shields.io/badge/Level-CCNA%20%2F%20CCNP-red?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completed-success?style=for-the-badge)

</div>

---

## 📌 Project Overview

EIGRP doesn't count hops. It builds a **composite metric** from **bandwidth** and **delay**. This lab shows exactly how the number in the routing table, **`[90/2172416]`**, is produced, by calculating it **by hand** and matching it to what the router displays.

Two Cisco 2911 routers (**R1**, **R2**) are connected by a serial link (**1.1.1.0/24**), each with its own LAN. R1's route to the remote LAN **192.168.2.0/24** is the one we dissect:

```
D    192.168.2.0/24 [90/2172416] via 1.1.1.2, 00:01:00, Serial0/0/0
```

> 🎯 **Result:** `256 × (6476 + 2010) = 2,172,416` ✅ — identical to the router output.

---

## 🎯 Objectives

- ✅ Learn the **EIGRP composite metric formula** (default K-values)
- ✅ Identify the **lowest bandwidth** along the path
- ✅ **Sum the delays** of the interfaces on the path
- ✅ Calculate the metric **manually** and match `show ip route`
- ✅ Understand **which interfaces count** in the calculation
- ✅ Understand **why integer truncation** matters (6476.68 → 6476)
- ✅ Learn how `bandwidth` and `delay` commands change path selection

---

## 🗺️ Network Topology

```
   192.168.1.0/24                                                  192.168.2.0/24
   BW 100 Mb · delay 10                                      BW 100 Mb · delay 10

PC1 ─┐                      Serial link                              ┌─ PC4
PC2 ─┼─ Switch0 ── Gi0/0 ┌──────┐ Se0/0/0 ═════ Se0/0/0 ┌──────┐ Gi0/0 ── Switch1 ─┼─ PC5
PC3 ─┘  (2950)    .1     │  R1  │ .1  1.1.1.0/24     .2 │  R2  │  .1    (2950)      └─ PC6
                        │ 2911 │  BW 1544 Kbps         │ 2911 │
                        └──────┘  delay 2000           └──────┘
```

> 📷 Add your screenshot: `![Topology](images/topology.png)`

---

## 📋 IP Addressing & Interface Values

| Device | Interface | IP Address | Bandwidth | Delay (tens of µs) |
|:------:|:---------:|:----------:|:---------:|:------------------:|
| **R1** | Gi0/0 | 192.168.1.1/24 | 100,000 Kbps (100 Mb) | **10** |
| **R1** | Se0/0/0 | 1.1.1.1/24 | **1,544 Kbps** | **2000** |
| **R2** | Gi0/0 | 192.168.2.1/24 | 100,000 Kbps (100 Mb) | **10** |
| **R2** | Se0/0/0 | 1.1.1.2/24 | **1,544 Kbps** | **2000** |

> ⏱️ Delay in EIGRP is measured in **tens of microseconds**. `show interfaces` prints it in microseconds: `2000 × 10 = 20,000 µs` and `10 × 10 = 100 µs`.

---

## 📐 The EIGRP Metric Formula

With the **default K-values** (K1 = 1, K3 = 1, K2 = K4 = K5 = 0), the full formula collapses to:

```
┌──────────────────────────────────────────────────────────────┐
│   Metric  =  256 × ( 10⁷ ÷ lowest bandwidth (Kbps)           │
│                      +  total delay (tens of µs) )           │
└──────────────────────────────────────────────────────────────┘
```

| Component | Rule |
|:----------|:-----|
| **Bandwidth** | Use the **lowest** bandwidth on the whole path (the bottleneck) |
| **Delay** | **Add up** the delay of every link on the path |
| **Divide** | `10,000,000 ÷ bandwidth` — result is **truncated** to an integer |
| **Multiply** | Final value is multiplied by **256** |

---

## 🧮 Step-by-Step Calculation (R1 → 192.168.2.0/24)

### 1️⃣ Find the lowest bandwidth on the path

| Link on the path | Bandwidth |
|:-----------------|:---------:|
| R1 Se0/0/0 → R2 (serial) | **1,544 Kbps** ◄── lowest |
| R2 Gi0/0 (LAN 2) | 100,000 Kbps |

➡️ **Bandwidth used = 1544 Kbps**

### 2️⃣ Add the delays on the path

| Link | Delay |
|:-----|:-----:|
| Serial link (Se0/0/0) | 2000 |
| LAN 2 link (R2 Gi0/0) | 10 |
| **Total** | **2010** |

### 3️⃣ Apply the formula

```
Metric = 256 × ( 10,000,000 / 1544 )  +  256 × ( 2000 + 10 )
       = 256 × ( 6476 )               +  256 × ( 2010 )
       = 1,657,856                    +  514,560
       = 2,172,416   ✅
```

> 🔎 **Truncation:** `10,000,000 ÷ 1544 = 6476.68…` → EIGRP uses **6476** (integer division, no rounding).

### 4️⃣ Compare with the router

```
R1# show ip route
D    192.168.2.0/24 [90/2172416] via 1.1.1.2, 00:01:00, Serial0/0/0
                         └─────┘
                     matches the hand calculation ✅
```

| Field | Meaning |
|:------|:--------|
| **D** | EIGRP internal route |
| **90** | Administrative Distance (internal EIGRP) |
| **2172416** | EIGRP composite metric |
| **via 1.1.1.2** | Next hop = R2 |

---

## 🔍 Which Delays Count? (Common Confusion)

For R1's route to **192.168.2.0/24**:

```
R1 Gi0/0 ✖     R1 Se0/0/0 ✔ ─────────► R2 Se0/0/0      R2 Gi0/0 ✔ ─► 192.168.2.0/24
(not on path)  delay 2000                              delay 10
                         └──────────── total delay = 2010 ────────────┘
```

- ✅ R1's **outgoing serial link** (2000)
- ✅ The **destination LAN interface** on R2 (10)
- ❌ R1's **own LAN interface** is *not* on the path to the remote network, so it's not added

The result is **symmetric**: R2's route to 192.168.1.0/24 also has metric **2,172,416**.

---

## 🧪 Cross-Check — Every Number in the EIGRP Series

The same formula explains the values from the other labs in this series:

| Network / Path | Lowest BW | BW term | Delay term | **Metric** |
|:---------------|:---------:|:-------:|:----------:|:----------:|
| Connected LAN (Gi0/0 only) | 100,000 | 100 | 10 | **28,160** |
| Connected serial 1.1.1.0/24 | 1544 | 6476 | 2000 | **2,169,856** |
| Remote LAN over serial (this lab) | 1544 | 6476 | 2010 | **2,172,416** |
| Remote LAN, serial **1000 Kbps** | 1000 | 10,000 | 2010 | **3,074,560** |
| Remote LAN, serial **500 Kbps** | 500 | 20,000 | 2010 | **5,634,560** |

```
Check 28,160   : 256 × (100 + 10)     = 28,160
Check 2,169,856: 256 × (6476 + 2000)  = 2,169,856
Check 3,074,560: 256 × (10000 + 2010) = 3,074,560
Check 5,634,560: 256 × (20000 + 2010) = 5,634,560
```

> 💡 The **Reported Distance (RD) 28160** seen in topology tables is simply the neighbor's own cost to its connected LAN.

---

## 🛠️ Verify the Inputs on the Router

```bash
show interfaces serial 0/0/0         # BW 1544 Kbit, DLY 20000 usec
show interfaces gigabitEthernet 0/0  # BW 100000 Kbit, DLY 100 usec
show ip route 192.168.2.0            # metric in the routing table
show ip eigrp topology               # FD / RD per route
show ip eigrp topology 192.168.2.0/24   # vector metric: min bandwidth, total delay (output detail varies in Packet Tracer)
show ip protocols                    # K-values: K1=1, K2=0, K3=1, K4=0, K5=0
```

### 🔧 Changing the metric inputs

```bash
interface Serial0/0/0
 bandwidth 1000        # Kbps — affects the METRIC only, not the real speed
 delay 2000            # tens of microseconds
```

> ⚠️ Changing `bandwidth` or `delay` changes **path selection**, not the physical link. It's the standard way to steer EIGRP traffic.

---

## 🐍 Bonus: Quick Metric Calculator (Python)

```python
def eigrp_metric(min_bandwidth_kbps: int, delays_tens_usec: list[int]) -> int:
    """Classic EIGRP metric with default K-values (K1=K3=1)."""
    bandwidth_term = 10_000_000 // min_bandwidth_kbps   # integer division (truncation)
    return 256 * (bandwidth_term + sum(delays_tens_usec))

# This lab: serial 1544 Kbps (delay 2000) + remote LAN (delay 10)
print(eigrp_metric(1544, [2000, 10]))   # 2172416
print(eigrp_metric(1000, [2000, 10]))   # 3074560
print(eigrp_metric(500,  [2000, 10]))   # 5634560
```

---

## 🧠 Core Concepts

| Term | Meaning |
|:-----|:--------|
| **Composite metric** | Single number built from bandwidth + delay (default) |
| **K-values** | Weights deciding which factors count (default K1 = K3 = 1) |
| **Bandwidth (min)** | The bottleneck link along the entire path |
| **Delay (sum)** | Total of all link delays along the path |
| **FD** | Feasible Distance = the total metric from this router |
| **RD** | Reported Distance = the metric the neighbor advertises |
| **AD (90)** | Trust level of internal EIGRP routes (not part of the metric) |

### 🔹 Why not hop count?
EIGRP prefers a **fast, low-latency** path even if it has more hops — far more realistic than RIP's hop count.

### 🔹 Load and reliability
K2, K4 and K5 are **0** by default, so **load and reliability are ignored** — the metric stays stable and doesn't flap with traffic.

### 🔹 Modern note
On newer IOS, **EIGRP named mode** can use **wide (64-bit) metrics**, so very fast links (10 Gbps+) are distinguished correctly. The classic formula in this lab is what CCNA/CCNP exams test.

---

## 🎓 Key Takeaways

1. 🔸 Default EIGRP metric = **256 × (10⁷ ÷ min bandwidth + total delay)**
2. 🔸 Use the **lowest bandwidth** on the path — **add up** the delays
3. 🔸 `10⁷ ÷ 1544 = 6476.68` is **truncated to 6476**
4. 🔸 Serial delay = **2000**, LAN delay = **10** → total **2010**
5. 🔸 `256 × 6476 + 256 × 2010 = 1,657,856 + 514,560 = 2,172,416`
6. 🔸 The hand-calculated value **matches** `show ip route`: `[90/2172416]`
7. 🔸 **Only interfaces on the path** count — R1's own LAN isn't part of the route to LAN 2
8. 🔸 `bandwidth` / `delay` commands change the metric (and therefore path choice), **not link speed**
9. 🔸 The same formula explains FD/RD values in every other EIGRP lab

---

## 🧪 Challenges

1. Set `bandwidth 1000` on Se0/0/0 and predict the new metric **before** checking the router
2. Change the serial `delay` to 1000 — how much does the metric drop? (Hint: 256 × 1000)
3. Add a second serial link and calculate the metric for each path
4. Calculate the metric to **1.1.1.0/24** from R2 and verify it
5. Find the lowest-bandwidth link in a 3-router chain and predict the metric end to end

---

## 📁 Repository Structure

```
📦 EIGRP-Metric-Calculation
 ┣ 📜 README.md
 ┣ 📂 images
 ┃ ┣ 🖼️ topology-with-calculation.png
 ┃ ┗ 🖼️ r1-show-ip-route-metric.png
 ┣ 📂 scripts
 ┃ ┗ 🐍 eigrp_metric.py
 ┗ 📂 lab-file
   ┗ 📦 EIGRP-Metric-Calculation.pkt
```

---

## 🔗 Series Navigation

⏮️ **Start:** EIGRP Successor – Single Path
➡️ Related: Successor 2 Paths · Unequal-Cost Load Balancing & Variance · MD5 Authentication

`EIGRP` · `Composite Metric` · `Bandwidth` · `Delay` · `K-values` · `Feasible Distance` · `Path Selection` · `Routing Fundamentals`

---

## 👨‍💻 Author

**Maaz Khan**
CCNA-Certified Network & NOC Engineer

[![LinkedIn](https://img.shields.io/badge/LinkedIn-maazkhanms-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/maazkhanms)
[![GitHub](https://img.shields.io/badge/GitHub-maazkhanms-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/maazkhanms)

---

<div align="center">

⭐ **If this lab helped you, star the repository!** ⭐

*Part of my ongoing Cisco Packet Tracer lab series — routing, switching, security & more.*

</div>
