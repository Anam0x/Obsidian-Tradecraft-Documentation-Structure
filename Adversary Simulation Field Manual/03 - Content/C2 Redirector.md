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

Red team operators and penetration testers use redirectors to control and restrict the flow of network traffic to their command‑and‑control (C2) infrastructure, while also obfuscating operator identities.

### Victim Network

Imagine an implant deployed on a victim network that communicates over an encrypted channel (typically HTTPS) to one or more redirectors hosted in a public cloud platform.

### Public Cloud Network

A redirector must not expose any operator‑identifying data, since it resides outside the operator’s controlled environment[^1].

A redirector can take many forms:
* [Virtual machine (VM)](https://csrc.nist.gov/glossary/term/virtual_machine) in AWS, Azure, GCP, etc.
* [Content Delivery Network (CDN)](https://csrc.nist.gov/glossary/term/content_delivery_networks) endpoint or edge function
* Serverless computing: [AWS Lambdas](https://aws.amazon.com/lambda/), [Azure Functions](https://learn.microsoft.com/en-us/azure/azure-functions/functions-overview), [Cloudflare Workers](https://workers.cloudflare.com/)
* [Platform-as-a-Service (PaaS)](https://csrc.nist.gov/glossary/term/platform_as_a_service) container with custom proxy logic

Operators should filter out unwanted traffic (e.g., scanners, crawlers, or blue‑team reconnaissance) by deploying a lightweight reverse proxy (e.g., Apache, NGINX) with rules evaluating:
* `User-Agent` HTTP headers
* Cookie values
* URI path patterns or file extensions
* Query‑string parameters

Suspicious requests can be dropped, redirected to a honeypot, or served benign content[^2]. Other solutions for filtering unwanted traffic exist, such as Web Application Firewalls (WAFs) that block known scanning patterns, automated exploit attempts, and generic bot traffic before it ever hits the redirector logic. When combined with DDoS protection services (e.g., AWS Shield, Azure DDoS Protection), this setup improves survivability against both opportunistic and targeted takedowns. Rate limiting by IP or region can further restrict exposure.

Using a valid, trusted certificate (e.g., via Let’s Encrypt) ensures traffic blends in with typical enterprise HTTPS traffic and avoids certificate errors or indicators that could alert defenders[^3]. In some cases, using mutual TLS (mTLS) or client certificate authentication can help authenticate implants or operator sessions explicitly. Certificates should be renewed automatically to reduce operational friction and downtime.

Operators may use more than one redirector to improve scaling and failover redundancy. Auto-scaling groups (for VMs) or concurrency-based scaling (for serverless functions) allow redirectors to handle unexpected load or distributed scans without crashing. Deploying redundant redirectors across different regions reduces the risk of a single point of failure and makes takedown efforts by defenders more challenging. Operators should ensure redirectors can gracefully fail over without implant downtime.

### Operator Network & Additional Considerations

Finally, operators establish an encrypted reverse SSH tunnel (or VPN tunnel) from their C2 team server to the redirector. Directly connecting back from the redirector to the C2 server is discouraged, as it would require persistent private keys on the redirector and open inbound access from the public cloud[^4].

Operators must continuously monitor redirector uptime and performance to avoid service interruptions. Cloud-native monitoring tools (e.g., AWS CloudWatch, Azure Monitor) can track HTTP status codes, error rates, and latency. Implementing periodic health checks can detect if a redirector has been taken offline or misconfigured. Consider using separate out-of-band alerting channels to notify operators if availability drops or unexpected spikes in traffic occur[^5].

Operators can deploy multiple redirectors in a chain to further obscure the true C2 infrastructure. For example, implant traffic may first reach a CDN edge worker, then forward to a second cloud-based VM, and finally reach the true C2 server. This multi-hop architecture can frustrate defenders' tracing efforts and force them to follow multiple layers of infrastructure. However, chaining increases operational complexity and requires careful tunnel and credential management to avoid introducing OPSEC risks[^6].

Redirectors, like most C2 infrastructure, should be treated as ephemeral assets; after an operation concludes, all redirectors, keys, client data, and configurations should be securely destroyed. Automate the creation and teardown of redirectors and other assets via Infrastructure as Code or cloud automation tools to reduce the risk of stale assets that could later be discovered by defenders and to reduce unnecessary hosting costs.

## Graphic

***
## Resources:

| Hyperlink | Info |
| --------- | ---- |
|           |      |

[^1]: 

***

*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>  
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
