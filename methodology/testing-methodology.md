# Testing Methodology

## Assessment Type

Black-box network security assessment.

## Objective

The assessment was performed to identify exposed network services
and determine whether the Internet-facing DNS service allowed
unauthorized recursive resolution.

The assessment also evaluated whether DNS responses were
substantially larger than corresponding requests.

---

## Scope

The assessment focused on:

- Network service enumeration
- DNS service identification
- Recursive DNS behavior
- External-domain resolution
- DNS response flags
- Negative DNS responses
- DNSSEC-related responses
- Packet-level traffic analysis
- Response/request size comparison
- Security impact assessment

---

## Phase 1 — Network Enumeration

TCP services were enumerated using Nmap.

Example:

```bash
nmap -sV -sC -O -A TARGET_IP
