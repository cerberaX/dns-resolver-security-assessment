# DNS Open Recursive Resolver & Response Amplification

> Black-box network security assessment of an Internet-exposed DNS service.

## Overview

This repository documents a black-box security assessment of an Internet-facing DNS service.

The assessment identified an Internet-accessible recursive DNS resolver and measured DNS response amplification using controlled DNS queries and real packet capture.

The assessment confirmed that the DNS service:

- Accepted DNS queries over UDP/53.
- Successfully resolved external Internet domains.
- Advertised recursive resolution through the `RA` DNS flag.
- Returned DNS responses significantly larger than corresponding requests.
- Produced a maximum measured response/request ratio of **5.68×**.
- Demonstrated measurable amplification across multiple DNS record types.

No third-party systems were targeted.

No source-IP spoofing was performed.

No denial-of-service traffic was generated.

---

## Assessment Summary

| Attribute | Details |
|---|---|
| Assessment Type | Black-box network security assessment |
| Primary Asset | Internet-facing DNS service |
| Primary Finding | Internet-exposed recursive DNS resolver |
| Security Category | DNS Misconfiguration / Infrastructure Security |
| Severity | High |
| Status | Confirmed |
| Primary Protocol | DNS / UDP |
| Primary Port | UDP/53 |
| Maximum Measured Amplification | **5.68×** |
| Testing Approach | Controlled diagnostic testing |
| Destructive Testing | Not performed |
| Source-IP Spoofing | Not performed |
| Third-Party Targeting | Not performed |

---

# Finding: Internet-Exposed Recursive DNS Resolver

**Severity:** High

**Category:** DNS Misconfiguration / Infrastructure Security

**Status:** Confirmed

## Description

The assessed DNS service was accessible from an Internet-facing interface and was capable of performing recursive DNS resolution for external domains.

A recursive resolver accepts DNS queries and obtains answers on behalf of clients. When recursion is unintentionally exposed to untrusted Internet clients, the service may be abused by unauthorized parties.

The finding was confirmed through a combination of:

1. Network service enumeration.
2. External DNS resolution testing.
3. DNS response flag analysis.
4. `RA` flag observation.
5. NXDOMAIN testing.
6. Packet-level DNS traffic capture.
7. Response/request size measurements.

---

# Technical Evidence

## UDP/53 Exposure

Network enumeration identified DNS over UDP/53:

```text
53/udp open domain
```


TCP/53 was also observed:
```text
53/tcp open domain
```
The primary finding concerns recursive DNS functionality accessible through the Internet-facing DNS service.
```text
Recursive DNS Resolution

The resolver successfully processed queries for external domains, including:

google.com
microsoft.com
cloudflare.com
wikipedia.org
example.com
```

Example:
```text
dig @TARGET_IP google.com A

Observed response characteristics:

status: NOERROR
flags: qr rd ra

The RA flag indicates that the server advertises recursive DNS availability.
```
The combination of:

Internet accessibility,
successful external-domain resolution,
and the RA flag

provided strong evidence that recursive DNS functionality was available to external clients.

External Domain Resolution

The following queries were successfully tested:
```text
dig @TARGET_IP google.com A
dig @TARGET_IP microsoft.com A
dig @TARGET_IP cloudflare.com A
dig @TARGET_IP wikipedia.org A
dig @TARGET_IP example.com A
```
Example response:
```text
SERVER: TARGET_IP#53
status: NOERROR
flags: qr rd ra
```
This demonstrated that the service was capable of resolving domains outside a locally controlled DNS namespace.

NXDOMAIN Testing

Negative DNS responses were also tested using non-existent domain names.

Example:
```text
dig @TARGET_IP nonexistent-random-domain.example A

Observed:

status: NXDOMAIN

This confirmed that the resolver was processing normal DNS queries and negative responses.

+norecurse Test

The following query was also performed:

dig @TARGET_IP cloudflare.com A +norecurse

The server responded while advertising:

RA

The +norecurse result was not treated as the sole evidence of open recursion.

The primary conclusion was based on the combined evidence of:

Internet accessibility;
successful resolution of external domains;
RA flag observation;
DNS query/response behavior;
packet-level traffic analysis.
DNS Response Amplification
Measurement Method

DNS traffic was captured using tcpdump:

sudo tcpdump -ni any -s0 -vv 'host TARGET_IP and udp port 53'

Request and response packet sizes were extracted from the captured traffic.

The observed response/request ratio was calculated as:

Amplification Ratio = Response Size / Request Size
```
The measurements represent actual captured IP packet lengths rather than estimated application-layer DNS payload sizes.

Measured Results
```text
DNS Query	            Request	Response	    Amplification
A example.com	        57 B	    300 B	    5.26×
AAAA example.com	        57 B	    324 B	    5.68×
MX example.com	        57 B	    283 B	    4.96×
NS example.com	        57 B	    140 B	    2.46×
TXT example.com	        57 B	    181 B	    3.18×
DNSKEY cloudflare.com	83 B	    431 B	    5.19×
DS cloudflare.com	    83 B	    319 B	    3.84×
A cloudflare.com	        83 B	    303 B	    3.65×
```

Maximum Measured Amplification

The highest observed response/request ratio was measured for:

AAAA example.com

Captured sizes:

Request:  57 bytes
Response: 324 bytes

Calculation:

324 / 57 = 5.68×

Therefore:

Maximum observed amplification: 5.68×

The minimum observed ratio in the tested set was:

2.46×
DNSSEC Observation

A DNSSEC-related query was tested using:

dig @TARGET_IP cloudflare.com DNSKEY +dnssec

Packet capture showed:

Request:  83 bytes
Response: 431 bytes

Calculation:

431 / 83 = 5.19×

This demonstrated that DNSSEC-related responses can also produce substantially larger responses than their corresponding requests.

Security Impact

An Internet-accessible recursive DNS resolver can potentially be abused as a reflector in DNS reflection/amplification attacks.

Simplified attack model:

                    DNS Request
Attacker ─────────────────────────────►
                                      │
                                      ▼
                              Recursive DNS
                                 Resolver
                                      │
                                      │ Larger
                                      │ DNS Response
                                      ▼
                                  Potential
                                   Victim

Potential impact includes:

Abuse complaints associated with the organization's infrastructure.
IP reputation degradation.
Increased DNS server resource consumption.
Increased network bandwidth utilization.
Potential contribution to reflection/amplification attacks.
Potential operational impact on other services sharing the same infrastructure.
Testing Limitations

The assessment intentionally did not perform active reflection or denial-of-service testing.

The following activities were not performed:

Source-IP spoofing.
Third-party victim targeting.
Volumetric DNS traffic generation.
Denial-of-service testing.
Stress testing.
Service exhaustion testing.

Therefore, this assessment does not claim that a real DNS reflection DDoS attack was successfully executed.

The confirmed security condition is:

Internet-accessible recursive DNS functionality combined with measured DNS response amplification.

Risk Assessment

The combination of publicly accessible recursion and measurable response amplification increases the risk that the service could be abused by unauthorized parties.

The principal security concern is not simply that DNS responses are larger than requests. The concern is that an Internet-accessible recursive resolver may provide an attacker with infrastructure that can participate in DNS reflection/amplification activity.

Recommendation
Primary Recommendation

If recursive DNS is not required to be publicly accessible:

Disable Internet-facing recursive DNS access.
Restrict recursive queries to trusted internal networks.
Restrict recursive DNS access to approved VPN ranges where applicable.
Implement DNS ACLs for trusted clients.
Apply firewall restrictions to UDP/53 and TCP/53 where appropriate.
Separate authoritative DNS and recursive resolver functionality where possible.
If Public DNS Access Is Required

If the service has a legitimate requirement to provide DNS functionality to external clients, recursion should still be restricted.

Recommended controls include:
```text
Explicit source-address ACLs.
Firewall-based access restrictions.
Separate authoritative and recursive DNS services.
DNS query-rate monitoring.
Anomaly detection.
Logging and alerting for unusual query volumes.
Periodic configuration review.
Additional Service Enumeration
```
The assessment also identified the following services.
```markdown
Port     	Service	    Observation
21/tcp    	FTP	        No confirmed exploitable vulnerability
53/tcp 	    DNS        	Recursive DNS exposure identified
53/udp	    DNS	        Recursive DNS exposure identified
80/tcp	    HTTP	        Redirect observed
443/tcp	    HTTPS       	No confirmed high-severity finding
2000/tcp	    MikroTik    Bandwidth Test	Internet exposure observed
2022/tcp	    SSH	        No confirmed exploitable vulnerability
8080/tcp	    Huly	        No confirmed authentication bypass/account takeover
```
Service exposure alone does not constitute a vulnerability.

Only issues supported by sufficient evidence were classified as confirmed findings.
```markdown
Evidence Confidence
Observation	                        Evidence	                      Confidence
UDP/53 exposed                      Nmap	                          High
TCP/53 exposed	                    Nmap	                          High
DNS resolver activ                  dig	                          High
External domains resolved	        dig	                          High
Recursive DNS available	            External resolution + RA      High
DNS response larger than request	    tcpdump                       High
2.46×–5.68× measured ratios	        Packet capture                High
DNSSEC response amplification	    Packet capture                High
Potential reflection/amplification  Combined evidence	          High
exposure	
Real DDoS performed	                Not tested	                  Not established
Source-IP spoofing	                Not tested	                  Not established
```

Methodology
```markdown
The assessment followed a controlled black-box network security testing methodology.

Phase 1 — Service Enumeration

Network services were identified using Nmap.

Example:

nmap -sV -p21,53,80,443,2000,2022,8080 TARGET_IP

UDP DNS exposure was separately assessed:

sudo nmap -sU -p53 --reason TARGET_IP
Phase 2 — DNS Behavior Analysis

DNS functionality was tested using dig.
```
Examples:
```bash
dig @TARGET_IP google.com A
dig @TARGET_IP microsoft.com A
dig @TARGET_IP cloudflare.com A
dig @TARGET_IP wikipedia.org A
dig @TARGET_IP example.com A
```
Phase 3 — Recursive Resolution Validation

Recursive behavior was evaluated using:

External domain resolution.
DNS response status.
DNS response flags.
RA observation.
Negative DNS responses.
+norecurse behavior.
Phase 4 — Packet Capture

DNS traffic was captured using:
```bash
sudo tcpdump -ni any -s0 -vv 'host TARGET_IP and udp port 53'
```
Captured packets were used to determine request and response sizes.

Phase 5 — Amplification Analysis

For each selected query:

Amplification Ratio =
Response IP Packet Size /
Request IP Packet Size

The resulting measurements were compared across multiple DNS record types.

Phase 6 — Risk Assessment

The confirmed technical condition was evaluated for:

Security impact.
Abuse potential.
Operational impact.
Network exposure.
Remediation feasibility.

No destructive testing was performed.

Tools

The following tools were used during the assessment:

Nmap
dig
tcpdump
Wireshark
Linux networking utilities
Evidence

Supporting evidence is available in the repository:

evidence/
├── dns-recursion.txt
├── nmap.txt
└── packet-analysis.txt

Visual evidence:

screenshots/
├── 01-dns-recursion.png
├── 02-nmap-udp53.png
├── 03-tcpdump-capture.png
└── 04-amplification-calculation.png

The complete sanitized assessment report is available in:

report/dns-open-recursion-assessment.pdf
Repository Structure
security-assessment-dns-open-recursion/
│
├── README.md
│
├── report/
│   └── dns-open-recursion-assessment.pdf
│
├── evidence/
│   ├── dns-recursion.txt
│   ├── nmap.txt
│   └── packet-analysis.txt
│
├── screenshots/
│   ├── 01-dns-recursion.png
│   ├── 02-nmap-udp53.png
│   ├── 03-tcpdump-capture.png
│   └── 04-amplification-calculation.png
│
└── methodology/
    └── testing-methodology.md
Key Result

The primary confirmed security finding was an Internet-accessible recursive DNS resolver.

Across the tested DNS queries, measured response/request ratios ranged from:

2.46× — 5.68×

The highest observed result was:

AAAA example.com


57 B → 324 B


5.68×

The finding should be remediated by restricting recursive DNS access to trusted networks and preventing unauthorized Internet clients from using the service as a recursive resolver.

Disclaimer

This repository contains a sanitized case study derived from an authorized security assessment.

Sensitive identifiers, client information, credentials, private infrastructure details, and other potentially identifying information have been removed or replaced.

The published material is intended to demonstrate security assessment methodology, technical analysis, evidence collection, and professional reporting practices.

No third-party systems were targeted.

No denial-of-service attack was performed.

No source-IP spoofing was performed.
