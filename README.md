# EdgeConnect SD-WAN Tunnel Calculator

> **A precision tunnel budget estimator for HPE Aruba Networking EdgeConnect SD-WAN deployments**

[![Live Demo](https://img.shields.io/badge/Live%20Demo-GitHub%20Pages-orange?style=for-the-badge&logo=github)](https://l00pback0.github.io/edgeconnect-tunnel-calculator)
[![Version](https://img.shields.io/badge/Version-4.3-blue?style=for-the-badge)]()
[![HTML](https://img.shields.io/badge/Built%20With-HTML%2FJS-informational?style=for-the-badge&logo=html5)]()
[![No Dependencies](https://img.shields.io/badge/Dependencies-None-brightgreen?style=for-the-badge)]()

---

## What Is This?

EdgeConnect appliances count **three distinct tunnel types** against their device tunnel budget — underlay, overlay, and passthrough. Most sizing tools only account for overlays, which leads to significantly undercounting real-world tunnel consumption.

This calculator models all three types accurately, accounting for:

- **Cross-Connect (CC) groups** — the N×M underlay matrix that forms when multiple WAN labels are in the same group (e.g. INET1 + INET2 cross-connecting with each other)
- **Business Intent Overlay (BIO) topology** — hub-spoke vs. full mesh, and which labels each BIO participates in
- **HA deployment modes** — Single Appliance, Traditional HA (independent WAN interfaces), and EdgeHA (shared via backplane) — each with correct same-site peer exclusion logic
- **Passthrough tunnels** — both WAN interface anchors (1 per label) and BIO×label entry points (1 per label per BIO)

Validated against live Orchestrator tunnel exports from a 12-appliance lab topology with 36/36 tunnel counts matching exactly across all three types.

---

## Quick Start

**Option 1 — Use the live hosted version:**
👉 [https://l00pback0.github.io/edgeconnect-tunnel-calculator](https://l00pback0.github.io/edgeconnect-tunnel-calculator)

**Option 2 — Run locally:**
Download `index.html` and open it in any modern browser. No server, no install, no dependencies.

---

## Features

### Tunnel Types Modeled

| Type | Formula | What It Represents |
|------|---------|-------------------|
| **Underlay** | CC group N×M per peer pair | IPsec/UDP transport tunnels — 1 per WAN label pair per peer relationship |
| **Overlay** | 1 per applicable BIO per peer | Application segmentation tunnels riding on top of underlays |
| **Passthrough (WAN)** | `label_count × (BIO_count + 1)` | Interface anchors + BIO×label entry points, local to each device |

### Configuration Options

- **WAN Labels** — define any number of labels with overlay topology scope (Any / Hub-Spoke Only) and CC group assignment (Groups 1–9)
- **Hub Gateway Groups** — sites with Single / Traditional HA / EdgeHA mode selection
- **Spoke Gateway Groups** — same HA mode options, site-centric input (tool derives appliance count automatically)
- **Business Intent Overlays** — up to 7 BIOs, each with Hub-Spoke or Mesh topology and per-label participation
- **SSE** — optional Security Service Edge passthrough tunnel overhead

### Results Panel

- Per-device tunnel total with capacity bar (green / amber / red)
- Type breakdown bars — underlay, overlay, passthrough at a glance
- Expandable detailed breakdown showing every peer relationship by tunnel type, including active WAN labels on each group
- Export to XLSX with Configuration, Summary, Breakdown, and Reference sheets

---

## How Tunnel Counting Works

### Cross-Connect Groups

Think of a CC group as a switchboard. When INET1 and INET2 are both in Group 1, every label on side A connects to every label on side B within that group — forming a full N×M underlay matrix.

```
Hub: INET1(G1), INET2(G1), MPLS(none)
Spoke: INET1(G1), MPLS(none)

Group 1 underlays: 2 × 1 = 2   (INET1↔INET1, INET2↔INET1)
None underlays:    1             (MPLS↔MPLS exact match)
Total underlays per pair: 3
```

### HA Mode Peer Counting

The calculator uses sites as the input unit and derives appliance count and peer logic automatically:

| HA Mode | Appliances/Site | Spoke↔Spoke Peers (per appliance) |
|---------|----------------|----------------------------------|
| Single Appliance | 1 | `(remote_sites) × 1` |
| Traditional HA | 2 | `(remote_sites) × 2` — both appliances at each remote site are independent peers |
| EdgeHA | 2 | `(remote_sites) × 2` + HA-link tunnel overhead |

Same-site appliances are always excluded from peering (the HA partner never counts as a spoke peer).

### Topology Scope

- **Any** labels participate in hub↔hub, hub↔spoke, and spoke↔spoke (mesh BIOs only) underlay and overlay formation
- **Hub-Spoke Only** labels are excluded from spoke↔spoke — typical for MPLS labels that don't reach all branch sites, or satellite links used only for hub backhaul

---

## Validation

The engine was validated against live tunnel exports from a 12-appliance HPE Aruba lab running ECOS with 3 BIOs (RealTime/mesh, LocalBreakout/hub-spoke, Default/hub-spoke) across East, West, and South regions.

```
Appliance       UL_exp  UL_act   UL_ok   OV_exp  OV_act   OV_ok   PT_exp  PT_act   PT_ok
──────────────────────────────────────────────────────────────────────────────────────────
BR1-EC1              9       9       ✓        8       8       ✓        6       6       ✓
BR2-EC1              5       5       ✓        7       7       ✓        3       3       ✓
BR2-EC2             10      10       ✓        7       7       ✓        9       9       ✓
BR3-EC1              8       8       ✓        5       5       ✓        6       6       ✓
BR3-EC2              4       4       ✓        5       5       ✓        3       3       ✓
BR4-EC1              9       9       ✓        5       5       ✓       12      12       ✓
BR4-EC2              9       9       ✓        5       5       ✓       12      12       ✓
BR5-EC1              4       4       ✓        3       3       ✓        6       6       ✓
DC1-EC1             19      19       ✓       15      15       ✓       12      12       ✓
DC1-EC2             19      19       ✓       15      15       ✓       12      12       ✓
DC2-EC1             34      34       ✓       21      21       ✓       12      12       ✓
DC3-EC1             16      16       ✓       12      12       ✓        6       6       ✓
```

**36/36 pass** across all appliances and all three tunnel types.

---

## Device Capacity Reference

| Model | Role | Max Tunnels |
|-------|------|------------|
| EC-US | Small Branch | 2,000 |
| EC-XS (9.2+) | Small Branch | 5,000 |
| EC-S-P | Large Branch | 5,000 |
| EC-M-H | Hub / DC | 10,000 |
| EC-L-H | Hub / DC | 10,000 |
| EC-XL-H | Hub / DC | 10,000 |

---

## Limitations & Notes

- **LAN-side passthroughs** are excluded — these are site-specific (depend on LAN interface count) and not topology-driven, so they can't be estimated from fabric configuration alone
- **WAN interface anchors** are included in passthrough count (1 per WAN label per device), since these are deterministic from the label inventory
- **EdgeHA HA-link tunnel count** is approximated as `⌈label_count / 2⌉` per appliance — the exact split depends on physical cabling at each site
- Estimates assume uniform label sets within each gateway group — create separate groups for sites with different WAN label combinations
- Fabric maximum: 5,000 gateways (ECOS 9.6+)

---

## Contributing

Found a discrepancy between the calculator output and your Orchestrator tunnel counts? Open an issue with:
- Your WAN label configuration and CC group assignments
- BIO topology and label participation
- The tunnel counts from your Orchestrator export (Underlay, Overlay, and Passthrough tunnel reports)

---

## Roadmap

The following features are planned for future releases. Contributions and feedback on prioritization are welcome via GitHub Issues.

### Regional Topology Support
Currently the calculator assumes a flat fabric where all spokes in a group can potentially peer with all hubs and all other spokes (subject to BIO topology rules). In real deployments, EdgeConnect fabrics are often segmented into regions — spokes only form tunnels with hubs and peers within their own region, while hub-to-hub tunnels span regions globally.

Planned additions:
- Define named regions and assign hub groups and spoke groups to them
- Spoke↔hub tunnel math scoped to matching region only
- Hub↔hub tunnels remain global (cross-region) as per standard EdgeConnect behavior
- Per-region tunnel summary in the results panel so you can see budget consumption by region alongside the per-device view

This is particularly important for large deployments where a single hub tier serves multiple geographic regions — without regional scoping, hub tunnel counts can be significantly overstated.

### Per-Spoke-Group BIO Assignment
Currently all BIOs apply to all gateway groups, which matches a simple flat fabric but doesn't reflect deployments where certain overlay policies only apply to specific site tiers. For example, a RealTime (voice/video mesh) BIO might only be relevant for Gold and Platinum sites, while Bronze and SOHO sites only participate in a Default hub-spoke overlay.

Planned additions:
- Checkbox matrix on each spoke group to select which BIOs that group participates in
- BIO participation affects both overlay count (fewer BIOs = fewer overlay tunnels) and passthrough count (`label_count × (active_BIO_count + 1)`)
- Results panel updated to reflect per-group BIO participation in the breakdown

This change will improve accuracy for tiered deployments and reduce false capacity warnings on lower-tier appliances that carry a lighter overlay footprint than the hub-centric view suggests.

---

## Disclaimer

This tool provides estimates for capacity planning purposes. Actual tunnel counts may vary based on site-specific configuration, ECOS version, and regional topology rules not captured in the calculator inputs. Always validate against Orchestrator tunnel reports for production sizing decisions.

---

*Built by L00pback0 using Claude.ai.  No, I am not a coder.  Just a former network engineer and current CSE at HPE Aruba Networking with ideas*
