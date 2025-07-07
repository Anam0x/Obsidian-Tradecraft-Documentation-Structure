---
aliases: 
tags:
  - 🏗️
primary categories:
  - "[[01 - Penetration Test]]"
  - "[[01 - Red Team]]"
secondary categories:
  - "[[02 - C2 Tradecraft & Profiles]]"
type: Infrastructure
---
# [[C2 Redirector]]

***
## Overview

Red team operators and penetration testers use redirectors to control and restrict the flow of network traffic to their [command and control (C2)](https://csrc.nist.gov/glossary/term/command_and_control) infrastructure, while also obfuscating their identities[^1].

### Victim Network

Imagine an implant deployed on a victim network that communicates over an encrypted channel (typically HTTPS) to one or more redirectors hosted in a public cloud platform.

The implant should blend into normal outbound traffic patterns, using standard ports (e.g., 443) and appearing similar to legitimate web, DNS, or API requests. Operators may design the implant to use common hostnames or mimic popular services to evade detection by network monitoring tools[^1][^].

Operators must carefully manage implant beaconing intervals and jitter to avoid generating suspicious network spikes or patterns. In some cases, defenders may analyze network request timing to detect covert channels, so operators should strike a balance between usability and stealth to maintain operational consistency[^2].

Implants often use a combination of short-haul and long-haul beacons. Short-haul beacons occur at relatively frequent intervals (e.g., every few minutes) and are typically used for regular check-ins or to receive small tasking updates. Long-haul beacons occur less frequently (e.g., every few hours, days, or weeks for long engagements) and are designed to reduce the risk of detection by minimizing network noise and blending into legitimate background traffic. By dynamically alternating between these modes, operators can maintain a responsive foothold while lowering the chance of detection during extended operations[^1].
### Public Cloud Network

A redirector must not expose any operator‑identifying data, since it resides outside the operator’s controlled environment.

A redirector can take many forms:
* [Virtual machine (VM)](https://csrc.nist.gov/glossary/term/virtual_machine) in AWS, Azure, GCP, etc.
* [Content Delivery Network (CDN)](https://csrc.nist.gov/glossary/term/content_delivery_networks) endpoint or edge function
* Serverless computing: [AWS Lambdas](https://aws.amazon.com/lambda/), [Azure Functions](https://learn.microsoft.com/en-us/azure/azure-functions/functions-overview), [Cloudflare Workers](https://workers.cloudflare.com/)
* [Platform-as-a-Service (PaaS)](https://csrc.nist.gov/glossary/term/platform_as_a_service) container with custom proxy logic
* And many more...[^1][^3]

Operators should filter out unwanted traffic (e.g., scanners, crawlers, or blue‑team reconnaissance) by deploying a lightweight reverse proxy (e.g., [Apache](https://httpd.apache.org/), [NGINX](https://nginx.org/)) with rules evaluating:
* [`User-Agent`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/User-Agent) HTTP headers
* [Cookie](https://csrc.nist.gov/glossary/term/cookie) values
* [URI](https://csrc.nist.gov/glossary/term/uniform_resource_identifier) path patterns or file extensions
* Query‑string parameters

Suspicious requests can be dropped or redirected to benign content[^1]. Other solutions for filtering include [Web Application Firewalls (WAFs)](https://csrc.nist.gov/glossary/term/web_application_firewall) that block known scanning signatures and automated exploit attempts before they reach the proxy. Combining these with [distributed denial of service (DDoS)](https://csrc.nist.gov/glossary/term/web_application_firewall) protection services (e.g., [AWS Shield](https://aws.amazon.com/shield/), [Azure DDoS Protection](https://learn.microsoft.com/en-us/azure/ddos-protection/ddos-protection-overview)) improves survivability against takedowns and large-scale scans. Rate limiting by IP or region further restricts exposure[^4].

Using a valid, trusted certificate (e.g., via [Let’s Encrypt](https://letsencrypt.org/)) ensures traffic blends in with typical enterprise HTTPS and avoids generating certificate errors that may tip off defenders. In some scenarios, [mutual TLS (mTLS)](https://csrc.nist.gov/glossary/term/mutual_tls) or client certificate authentication can help authenticate implants explicitly[^5]. Automating certificate renewal reduces operational risk and downtime[^3][^4].

Operators often deploy multiple redirectors for scalability and redundancy. [Auto-scaling](https://www.ibm.com/think/topics/autoscaling) groups (for VMs) or [concurrency-based scaling](https://www.toucantoco.com/en/glossary/automatic-concurrency-scaling.html) (for serverless functions) allow redirectors to handle unexpected load without crashing[^4]. Distributing redirectors across different regions mitigates single points of failure and complicates defender takedowns. Operators should ensure that redirectors can fail over gracefully without interrupting implant communication.

### Red Team Network & Additional Considerations

Operators typically establish an encrypted reverse [SSH](https://csrc.nist.gov/glossary/term/secure_shell_network_protocol) tunnel (or [VPN tunnel](https://csrc.nist.gov/glossary/term/tunnel_vpn)) from the C2 team server to the redirector[^1]. Directly connecting the redirector back to the C2 server is discouraged, as this would require storing sensitive private keys on the redirector and allow inbound access from the public cloud, increasing OPSEC risk.

Operators should continuously monitor redirector uptime and performance to avoid service interruptions[^3][^4]. Cloud-native monitoring tools (e.g., [AWS CloudWatch](https://aws.amazon.com/cloudwatch/), [Azure Monitor](https://learn.microsoft.com/en-us/azure/azure-monitor/fundamentals/overview)) can track HTTP status codes, error rates, and latency. Implementing periodic health checks detects if a redirector has been taken offline or misconfigured. Using out-of-band alerting (e.g., [Slack](https://slack.com/), [SMS](https://csrc.nist.gov/glossary/term/short_message_service), or custom [webhooks](https://help.make.com/webhooks)) can quickly notify operators of availability issues or unexpected traffic spikes.

Operators can also deploy chained redirectors to further obscure the true C2 infrastructure. For example, implant traffic may first reach a CDN edge worker, then pass through a cloud-based VM, before finally arriving at the C2 server. This multi-hop architecture frustrates defender traceback efforts but increases operational complexity and requires careful tunnel and credential management to avoid introducing OPSEC risks[^1].

Redirectors, like most C2 infrastructure, should be treated as disposable assets. After an operation, securely destroy redirectors, keys, client data, and configurations[^6]. Automating the creation and teardown of redirectors via [Infrastructure as Code (IaC)](https://csrc.nist.gov/glossary/term/infrastructure_as_code) or cloud automation tools reduces the risk of leftover assets that defenders might later discover and lowers hosting costs.

## Graphic


***
## Resources:

| Hyperlink                                                                                                                                                                   | Info                                                                                                                                        |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| ["Red Team Tutorial: Design and setup of C2 traffic redirectors" by *Dmitrijs Trizna*](https://ditrizna.medium.com/design-and-setup-of-c2-traffic-redirectors-ec3c11bd227d) | Medium blog post on C2 infrastructure with a focus on redirectors                                                                           |
| [Red Team Ops II - Zero-Point Security](https://training.zeropointsecurity.co.uk/courses/red-team-ops-ii)                                                                   | A continuation of ZPS's "Red Team Ops" course; one of the primary learning objectives is the maintenance and hardening of C2 infrastructure |

[^1]: https://ditrizna.medium.com/design-and-setup-of-c2-traffic-redirectors-ec3c11bd227d
[^2]: https://www.varonis.com/blog/jitter-trap
[^3]: https://infosecwriteups.com/mastering-c2-redirectors-a-red-blue-teamers-guide-3e1c6d34ecc8
[^4]: https://xbz0n.sh/blog/c2-redirectors
[^5]: https://byt3bl33d3r.substack.com/p/revisiting-cloudflare-workers-for 
[^6]: https://www.infosecinstitute.com/resources/penetration-testing/red-team-assessment-phases-completing-objectives/

***

*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>  
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
