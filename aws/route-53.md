# AWS Route 53 Notes

## What is Route 53?

Route 53 is AWS's scalable **Domain Name System (DNS)** web service. The name is a reference to **port 53**, the standard network port used for DNS traffic.

---

## Core Functions

### 1. Domain Registration
- Buy and manage domain names (e.g., `example.com`) directly through Route 53
- Supports hundreds of TLDs (.com, .org, .io, etc.)

### 2. DNS Routing
Translates human-readable domain names into IP addresses so browsers can load the correct resource.

### 3. Health Checking
Monitors the health of endpoints (servers, load balancers, etc.) and automatically reroutes traffic away from unhealthy ones.

---

## Routing Policies

| Policy | Description |
|---|---|
| **Simple** | Route traffic to a single resource |
| **Weighted** | Split traffic across resources by percentage (e.g., 80/20) |
| **Latency-based** | Route users to the lowest-latency AWS region |
| **Failover** | Redirect traffic automatically if primary resource goes down |
| **Geolocation** | Route based on the user's geographic location |
| **Geoproximity** | Route based on proximity to resources, with optional bias |
| **Multivalue Answer** | Return multiple healthy records to the client |

---

## Port 53 & DNS

- **Port 53** is the universally assigned port for DNS traffic (assigned by IANA)
- **UDP port 53** — used for most regular DNS lookups (fast, lightweight)
- **TCP port 53** — used when response is too large for UDP, or for zone transfers (copying DNS records between servers)

---

## Key Concepts

### Hosted Zones
A container for DNS records for a specific domain.

- **Public Hosted Zone** — resolves domain names on the public internet
- **Private Hosted Zone** — resolves domain names within one or more AWS VPCs (internal DNS)

### Record Types
| Record | Purpose |
|---|---|
| **A** | Maps domain to an IPv4 address |
| **AAAA** | Maps domain to an IPv6 address |
| **CNAME** | Alias for another domain name |
| **MX** | Mail exchange — routes email to mail servers |
| **TXT** | Text records — used for verification, SPF, DKIM |
| **NS** | Name server records |
| **SOA** | Start of Authority — metadata about the zone |
| **Alias** | AWS-specific — maps to AWS resources (ELB, CloudFront, S3) |

> **Note:** Alias records are Route 53-specific and can point to AWS resources like load balancers, CloudFront distributions, and S3 static websites — unlike CNAME, they work at the zone apex (e.g., `example.com`).

### TTL (Time to Live)
How long DNS resolvers cache a record before re-querying. Lower TTL = faster propagation of changes, but more DNS queries.

---

## Common Use Cases in Organizations

### Basic Website/App Hosting
Point a domain to AWS infrastructure (EC2, Load Balancer, S3, etc.)

### Multi-Region High Availability
Use latency-based routing to direct users to the nearest/fastest AWS region automatically.

### Disaster Recovery
Failover routing + health checks — Route 53 detects an unhealthy primary endpoint and automatically reroutes to a backup in seconds.

### Internal Microservices DNS
Private hosted zones allow services to discover each other by name (e.g., `payments-service.internal`) inside a VPC instead of hardcoded IPs.

### Blue/Green & Canary Deployments
Use weighted routing to gradually shift traffic during deployments:
- Start: 95% old version / 5% new version
- Verify → increase gradually → 100% new version
- Reduces deployment risk significantly

### CDN Integration (CloudFront)
Point domain to a CloudFront distribution via Alias record to serve content from global edge locations with low latency.

### Email Routing
Manage MX records to route email for a domain to mail providers (e.g., Google Workspace, Microsoft 365).

### SaaS Custom Domains
Automate custom domain management for customers via the Route 53 API (e.g., `app.customer.com` → your SaaS platform).

---

## Integration with AWS Services

Route 53 integrates natively with:
- **EC2** — point domains to instances
- **Elastic Load Balancer (ELB)** — distribute traffic across instances
- **CloudFront** — CDN with global edge locations
- **S3** — static website hosting
- **API Gateway** — custom domains for APIs
- **VPC** — private DNS via private hosted zones
- **CloudWatch** — health check alarms and monitoring

---

## Health Checks

Route 53 can monitor:
- HTTP/HTTPS endpoints
- TCP connections
- Other Route 53 records (calculated health checks)

Health checks can trigger:
- DNS failover (reroute traffic)
- CloudWatch alarms
- SNS notifications

---

## Pricing Model (High Level)

- **Hosted zones** — charged per hosted zone per month
- **DNS queries** — charged per million queries
- **Domain registration** — annual fee per domain (varies by TLD)
- **Health checks** — charged per health check per month

> Always check the [AWS Route 53 pricing page](https://aws.amazon.com/route53/pricing/) for current rates.

---

## Quick Reference: Route 53 vs Traditional DNS

| Feature | Route 53 | Traditional DNS |
|---|---|---|
| Global anycast network | ✅ | Varies |
| Health checks + failover | ✅ | Limited |
| AWS service integration | ✅ | ❌ |
| Latency/geo routing | ✅ | Limited |
| Private DNS (VPC) | ✅ | ❌ |
| SLA | 100% uptime SLA | Varies |

---

## Useful CLI Commands (AWS CLI)

```bash
# List hosted zones
aws route53 list-hosted-zones

# List records in a hosted zone
aws route53 list-resource-record-sets --hosted-zone-id <ZONE_ID>

# Create a record (using a JSON change batch)
aws route53 change-resource-record-sets \
  --hosted-zone-id <ZONE_ID> \
  --change-batch file://change-batch.json

# Check health of a health check
aws route53 get-health-check-status --health-check-id <ID>
```

---

## Resources

- [AWS Route 53 Documentation](https://docs.aws.amazon.com/route53/)
- [Route 53 Developer Guide](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/Welcome.html)
- [Route 53 Pricing](https://aws.amazon.com/route53/pricing/)
- [AWS DNS Best Practices](https://docs.aws.amazon.com/whitepapers/latest/hybrid-cloud-dns-options-for-vpc/hybrid-cloud-dns-options-for-vpc.html)
