# ICS Security: Defense and Mitigation for Schneider Modicon M340 PLC

> Practical implementation and validation of an inline Intrusion Prevention System (Suricata) protecting a vulnerable industrial PLC against multiple critical CVEs. Academic project, MSc in Informatics Engineering, University of Coimbra.

📄 **[Read the full technical report (PDF)](ISprojectFinal-3.pdf)** — 11 pages

---

## About this project

This was a hands-on project for the Industrial Systems Security course at the University of Coimbra (MSc in Informatics Engineering, 2025/2026), supervised by Prof. Tiago Cruz. We worked on a real Schneider Electric Modicon M340 PLC testbed — the kind of hardware found in water, energy, and transportation control systems — and built a defense-in-depth architecture around it.

The full implementation, validation, and analysis is documented in the report. This repository serves as a reference and case study; the testbed code itself is not published as it was tied to a controlled academic environment.

**Authors:** João Dias, André Teixeira
**Supervisor:** Prof. Tiago Cruz (CISUC — Cybersecurity Transversal Laboratory)

## What was done

We deployed **Suricata in inline IPS mode** as an active defense between the supervisory network and the PLC, wrote custom protocol-aware rules targeting specific CVEs, and complemented it with a Python-based File Integrity Monitoring system. Each rule was validated against a working exploit before being considered effective.

The work is mapped explicitly to **IEC 62443** (Zones & Conduits, Security Levels) and **NIST SP 800-82 Rev. 2** recommendations.

## Key results

| Metric | Result |
|---|---|
| Critical CVEs blocked | **5/5 (100%)** |
| OpenVAS reconnaissance traffic dropped | **67%** |
| Mean Time To Detect | **< 0.5 seconds** |
| Added network latency (inline IPS) | **~18 ms** |
| False positive rate (after tuning) | **0.3%** |
| IEC 62443 Security Level | **SL 0 → SL 2** |

## CVEs mitigated

| CVE | CVSS | Attack vector | Mitigation approach |
|---|---|---|---|
| CVE-2017-6017 | 7.5 | Modbus DoS via Function Code 0x5A — causes full PLC freeze | Inline drop rule on the malicious FC |
| CVE-2015-6490 | 10.0 | UMAS protocol takeover (FC 90) — remote logic upload, STOP/START | Protocol-aware Modbus function filtering |
| CVE-2020-7569 | 9.8 | Hardcoded FTP credentials embedded in firmware | Content filtering on FTP login pattern |
| CVE-2015-7937 | 7.5 | GoAhead web server buffer overflow | HTTP/80 blocking at IPS layer |
| — | 5.3 | SNMP community-based information disclosure | UDP/161 query filtering |

## Highlights

- **Protocol-aware detection.** Suricata's native Modbus parser was used to inspect Function Codes inside the payload, rather than relying on port-based filtering — essential for distinguishing legitimate HMI polling (FC 3) from malicious commands (FC 0x5A, FC 90).
- **Validated under load.** During an aggressive Greenbone OpenVAS scan, the IPS processed ~20 million packets across both interfaces while dropping 67% of reconnaissance traffic without legitimate packet loss.
- **File Integrity Monitoring.** A custom Python FIM script with SHA-256 baselines, scheduled via cron every 2 minutes, monitors `suricata.yaml` and `ics.rules` for tampering. The production design includes an automated PLC shutdown response.
- **Standards-driven.** Every defense decision is mapped to specific IEC 62443 requirements (SR 1.1, SR 3.1, SR 6.2, etc.) and the Purdue Model. The report includes a gap analysis for progression to SL 3.
- **Defense-in-depth thinking.** The report explicitly discusses residual risks (insider threat, encrypted channels reducing DPI visibility) and the limits of the approach.

## What I learned

- The difference between IDS and IPS is not academic — passive detection would have allowed the DoS exploit to reach the PLC and cause a physical reset every time.
- Modbus and UMAS are designed without authentication; protecting them requires understanding the protocol at the application layer, not just at the port level.
- Latency overhead from inline DPI (~18 ms in our setup) is well within ICS supervisory communication tolerances, but the trade-off has to be argued, not assumed.
- The hardest part of writing IPS rules is not blocking the attack — it's **not** blocking the legitimate HMI traffic. Initial false positive rate was 8.5%, reduced to 0.3% after subnet whitelisting.

## Standards referenced

- **IEC 62443-3-3** — Security Levels for Industrial Automation Systems
- **IEC 62443-3-2** — Zones and Conduits
- **NIST SP 800-82 Rev. 2** — Guide to ICS Security
- **Lockheed Martin Cyber Kill Chain** — defense mapping across the attack lifecycle

## License

The report is shared under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — feel free to cite or reference the work with attribution.

## Acknowledgments

Testbed and technical guidance provided by **Prof. Tiago Cruz** (Department of Informatics Engineering, University of Coimbra). Open-source tools used: Suricata (OISF), Greenbone OpenVAS, Scapy.
