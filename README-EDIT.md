# FireWALL-E: Automated IP Blocklist Aggregator 🤖

<h5 align="center">This repository contains an automatically maintained collection of IP sets and netsets (blocklists) aggregated from various sources across the internet. It is designed to be a reliable source for firewall blocking rules.</h5>

<h4 align="center">
  <a href="https://github.com/kraloveckey/ipsets-blocklist"><img src=".assets/firewall-e0.png" width=250 lt="FireWALL-E"></a>
</h4>

> [!NOTE]
> The idea for this repository comes from [blocklist-ipsets](https://github.com/firehol/blocklist-ipsets) made by FireHOL Project, that are no longer updated regularly. If you prefer not to use this version, you can check out the original.

This repository is updated once per day and includes a list of ipsets dynamically updated using the custom `firewall-e.sh` script, which is a heavily modified and optimized version of [FireHOL's update-ipsets](https://github.com/firehol/firehol/blob/master/sbin/update-ipsets) script.

This repository is self-maintained and operates automatically via a daily cron job. It pulls the latest threat intelligence, applies a custom whitelist, processes the data using [iprange](https://github.com/firehol/iprange), and commits the changes directly to this repository.

---

## Overview
- [FireWALL-E: Automated IP Blocklist Aggregator 🤖](#firewall-e-automated-ip-blocklist-aggregator-)
  - [Overview](#overview)
  - [Contributing](#contributing)
  - [Why do we need blocklists?](#why-do-we-need-blocklists)
  - [Using ipsets](#using-ipsets)
  - [Which ipsets to use?](#which-ipsets-to-use)
  - [Tactical value of open proxy blocklists](#tactical-value-of-open-proxy-blocklists)
- [List of ipsets 🔥](#list-of-ipsets-)

---

## [Contributing](CONTRIBUTING.md)

**[`^        back to top        ^`](#overview)**

> [!IMPORTANT]
> Your contributions and suggestions are heartily welcome. Please, check the [Guide](CONTRIBUTING.md) for more details.
>
> If you want to propose changes, just open an [issue](https://github.com/kraloveckey/venom/issues) or a [pull request](https://github.com/kraloveckey/venom/pulls).

---

## Why do we need blocklists?

**[`^        back to top        ^`](#overview)**

As the global digital infrastructure matures, the complexity and sophistication of cybercrime are scaling at an unprecedented rate. We have moved far beyond the era where legacy security stacks – standalone antivirus software, basic firewalls, and traditional Intrusion Detection/Prevention Systems (IDS/IPS) – were sufficient to keep malicious actors at bay. Today's threat landscape requires a much broader perspective.

The most critical paradigm shift in modern cyber warfare is that adversaries often do not intend to cause direct, visible damage to your systems or steal your proprietary data. Instead, they increasingly view your infrastructure as a disposable, tactical asset. Compromised networks are routinely hijacked to serve as silent proxies, botnet nodes, or spam relays to facilitate attacks against completely unrelated third parties. Because these attacks are highly distributed, dynamically routed, and originate from an ever-shifting pool of global IP addresses, identifying and mitigating them in isolation is practically impossible.

To effectively combat these distributed threats, operating in a silo is no longer an option. We must augment our localized security postures with shared global intelligence and collaborative experience. Fortunately, a dedicated community of security researchers, analysts, and threat-hunting teams works around the clock to monitor traffic, deploy honeypots, and pinpoint malicious infrastructure. The result of their labor is the continuous release of curated blocklists targeting compromised domains, URLs, and most crucially, toxic IP addresses.

In this project, our primary focus is strictly on IP addresses.

Integrating robust, community-driven IP blocklists at the outermost edge of your firewall is a foundational pillar of modern internet security. This proactive strategy functions as a form of digital herd immunity – allowing us to leverage the community's collective telemetry to preemptively drop traffic from known fraudsters, automated scanners, and exploit kits before they ever touch our internal services.

**Why aggregate this intelligence on GitHub?**

The decision to centralize these diverse threat feeds into a single GitHub repository is driven by three core engineering advantages:

1. `Frictionless Availability and Open Source Ethos`: These lists are compiled by teams dedicated to improving global internet security and are freely distributed across the web. Aggregating them here creates a reliable, single source of truth. (*Disclaimer*: While these feeds are publicly available, some may carry specific licensing constraints. Always verify the upstream policies of the original creators before deploying them in commercial environments).
2. `Streamlined Automation`: Manually parsing and updating dozens of disparate threat feeds is a logistical nightmare. GitHub provides an elegant, unified distribution mechanism. By simply configuring a scheduled `git pull` (e.g., via a cron job) on your edge devices, routers, or servers, you can instantly synchronize your entire infrastructure with the latest global threat intelligence in one command.
3. `Transparent Version Control and Auditing`: Git is the ultimate tool for tracking data over time. Hosting these lists here provides a granular, auditable history of the shifting threat landscape. Network administrators can effortlessly track the delta between commits, allowing them to see exactly when a specific IP or subnet was flagged as malicious and precisely when it was remediated and removed from the list.
   
---

## Using ipsets

**[`^        back to top        ^`](#overview)**

While integrating automated threat intelligence is highly effective, it requires strategic and deliberate implementation. Blindly dropping traffic based on external blocklists carries inherent operational risks. A misconfigured firewall rule can easily trigger a self-inflicted Denial of Service (DoS), inadvertently locking out legitimate users or critical business customers or even severing your own administrative access to the infrastructure.

Please deploy these feeds responsibly by adhering to the following core principles:

1. Before integrating any external feed into a production environment, take the time to visit the original maintainer's website. Understand their detection methodology, false-positive mitigation strategies, and data retention policies. By feeding their IP lists directly into your routing logic, you are implicitly trusting their operational accuracy. Know exactly who you are trusting to protect your perimeter.

2. Generating and maintaining high-fidelity threat intelligence requires immense computational resources, bandwidth, and human effort. Many of these dedicated research teams rely on community backing to sustain their operations. If you derive value from their data, consider supporting them through their donation models or upgrading to their premium, commercial-grade feeds to ensure the continued quality and longevity of their work.

3. Threat blocklists must be strictly applied at the absolute perimeter – specifically, the internet-facing WAN interface of your firewall. Exercise extreme caution regarding placement. Certain feeds intentionally include unroutable, private IP spaces (e.g., RFC 1918 addresses). If you mistakenly apply these specific blocklists to your internal interfaces (LAN, DMZ, or management VLANs), you will immediately blackhole your internal routing and irrevocably lock yourself out of your own hardware.

4. A proactive security posture must always be paired with a reliable failsafe. You must maintain a robust, static whitelist containing the IP addresses and CIDR subnets of your trusted infrastructure, essential business partners, and administrative endpoints. Your firewall hierarchy must be engineered so that explicit "Allow" rules strictly override any external blocklist logic.

> [!NOTE]
> To mitigate these risks, **FireWALL-E** has been engineered with built-in safeguards. The script natively parses our predefined whitelist file and strips those trusted IPs from all downloaded threat feeds before the final blocklists are generated and committed. This guarantees that your critical assets remain inherently immune to upstream false positives.

---

## Which ipsets to use?

**[`^        back to top        ^`](#overview)**

Selecting the appropriate threat intelligence feeds requires carefully balancing your organization's risk tolerance with operational availability. Blindly applying every available blocklist will inevitably lead to severe service disruption and false positives. Instead, adopt a layered, Zero Trust-aligned defense strategy.

Your baseline protection should begin at the absolute edge of your network with foundational, zero-tolerance intelligence. These high-fidelity feeds typically target globally recognized cybercrime syndicates, malware command-and-control infrastructure, and unroutable IP spaces. Because their false-positive rate is near zero, they are ideal for silently dropping toxic traffic at core routers before it consumes firewall state-table resources. Moving deeper into the perimeter, this baseline should be augmented with dynamic threat mitigation feeds that track active brute-force campaigns, credential-stuffing bots, and automated scanners. While these secondary blocklists are highly effective at shielding exposed administrative interfaces or application firewalls, they carry a slight risk of catching legitimate users due to dynamic IP churn, necessitating a robust whitelist and continuous log monitoring.

Beyond baseline protection, the deployment of more aggressive or context-dependent intelligence relies entirely on your specific business model. Threat actors heavily exploit public, unauthenticated infrastructure – such as open proxies and anonymization networks – to mask their origins and bypass rate limits. If you operate an enterprise authentication gateway or a financial platform where verifying true geographic origins is critical for fraud prevention, aggressively blackholing these obfuscation nodes is a highly effective tactic to strip attackers of their camouflage. However, applying these anonymity-control lists, along with crowdsourced heuristic feeds that flag generic spam or aggressive web scraping, requires extreme caution. In environments supporting privacy-conscious users or general public access, such strict rules will inevitably disrupt legitimate traffic and are often better utilized for enriching SIEM telemetry or protecting isolated honeypots rather than triggering automated connection drops.

## Tactical value of open proxy blocklists

**[`^        back to top        ^`](#overview)**

To be explicitly clear, the open proxy lists are not aggregated here to facilitate proxy shopping or anonymization routing.

If you analyze the underlying data and cross-reference open proxy feeds with active threat intelligence (such as `blocklist_de` or `stopforumspam`), you will immediately notice a massive intersection.

**This critical overlap proves a well-known operational tactic**: Threat actors heavily rely on public, unauthenticated open proxies to obfuscate their true origins while executing distributed attacks.

By proactively blackholing known anonymization infrastructure, you are effectively stripping attackers of their camouflage. If your systems fall under an active, distributed attack, preemptively blocking these open proxy networks can instantly neutralize a vast portion of the malicious traffic.

---

# List of ipsets 🔥

<h4 align="center">
  <a href="https://github.com/kraloveckey/ipsets-blocklist"><img src=".assets/firewall-e1.png" width=250 lt="FireWALL-E"></a>

**[`^        back to top        ^`](#overview)**
</h4>

