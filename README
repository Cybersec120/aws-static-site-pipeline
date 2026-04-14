







![Architecture Diagram](docs/architecture.jpg)
A production-ready static website delivery platform on AWS using **Amazon S3**, **Amazon CloudFront**, **Terraform**, and **GitHub Actions** with OIDC federation.

This project was not built by starting with services. It was built by starting with a customer problem.

---

## Table of Contents

- [The Story Behind the Solution](#the-story-behind-the-solution)
- [Customer Problem](#customer-problem)
- [Business Goal](#business-goal)
- [Discovery Framework](#discovery-framework)
- [Architecture Decision](#architecture-decision)
- [Architecture Diagram](#architecture-diagram)
- [Solution Overview](#solution-overview)
- [AWS Well-Architected Alignment](#aws-well-architected-alignment)
- [Repository Structure](#repository-structure)
- [CI/CD Pipeline](#cicd-pipeline)
- [Production Design Considerations](#production-design-considerations)
- [Prerequisites](#prerequisites)
- [How to Deploy](#how-to-deploy)
- [Terraform Variables](#terraform-variables)
- [Rollback Strategy](#rollback-strategy)
- [Security Notes](#security-notes)
- [Business Outcome](#business-outcome)
- [Future Enhancements](#future-enhancements)
- [Author Perspective](#author-perspective)
- [License](#license)

---

## The Story Behind the Solution

A customer came in with what sounded like a website issue, but it was really an operations problem.

Their public-facing site was hosted in a traditional way. Updates were manual. Someone had to log into the environment, upload files, and hope nothing broke. Performance was inconsistent for users in different regions. There was no clean deployment process, no standardized rollback approach, and too much of the environment depended on human effort.

The website itself was not complex — it was a static frontend workload. But the way it was being delivered made it fragile.

The client did not need more infrastructure. They needed the **right** infrastructure.

That shifted the conversation away from servers and toward architecture. Instead of maintaining web servers for a workload that did not need them, I designed a production-grade static website delivery platform using AWS managed services. The result was a solution that was faster, more secure, easier to operate, and significantly more aligned with cloud best practices.

---

## Customer Problem

The existing environment had several issues that were creating risk and slowing the team down:

- Deployments were manual and inconsistent, requiring direct administrator intervention
- The site performed poorly for geographically distributed users with no caching layer
- Rollback was not well defined — recovering from a bad deploy meant manual file restoration
- Server management created unnecessary operational overhead for a static workload
- The architecture exposed more infrastructure surface area than necessary
- The environment was not reproducible as code and could not be version-controlled
- The customer wanted a production-ready design, not a quick fix

---

## Business Goal

The customer wanted a platform that would:

- Support reliable, repeatable production deployments with zero manual intervention
- Improve global website performance through edge caching
- Reduce hosting and maintenance overhead by eliminating server management
- Integrate with a modern CI/CD workflow using version-controlled automation
- Align with AWS best practices and the Well-Architected Framework
- Create a cleaner operational model that scales without adding complexity

---

## Discovery Framework

Before designing the solution, I worked through a structured discovery model so the recommendation would be driven by requirements rather than assumptions.

### BATT-SOR Framework

| Phase | Focus | Key Question |
|-------|-------|-------------|
| **Business** | Problem definition | What problem are we solving, and why does it matter? |
| **Architecture** | Current state | What does the existing environment look like? |
| **Technical** | Requirements | What must the solution do? |
| **Threats** | Risk assessment | What security, availability, or operational risks exist? |
| **Scale** | Performance targets | What are the performance and growth expectations? |
| **Operations** | Ownership model | Who will run it, monitor it, and support it? |
| **Roadmap** | Constraints | What budget, timeline, and platform limits exist? |

This created a repeatable, professional intake process and confirmed that the workload was a strong fit for static delivery on AWS.

---

## Architecture Decision

After evaluating the workload, I determined that this site did **not** require:

- EC2-based web hosting or container orchestration
- Persistent compute for page rendering
- An overengineered application tier
- Long-lived credentials or static API keys

Because the workload was static, the architecture needed to optimize for simplicity, security, repeatability, performance, and cost efficiency. That led to the following design:

| Component | Purpose |
|-----------|---------|
| **Amazon S3** | Private static asset storage (no public access) |
| **Amazon CloudFront** | Global CDN with HTTPS, HTTP/2+3, and edge caching |
| **CloudFront OAC** | Origin Access Control — S3 only accessible via CloudFront |
| **GitHub Actions** | CI/CD automation triggered on push to `main` |
| **OIDC Federation** | Keyless AWS authentication — zero static credentials |
| **Terraform** | Infrastructure as code for reproducible deployments |

---

## Architecture Diagram

![Architecture Diagram](docs/architecture.svg)

---

## Solution Overview

At a high level, the solution works like this:

1. A developer pushes approved code to the `main` branch
2. GitHub Actions authenticates to AWS via OIDC federation (no stored keys)
3. The pipeline builds static assets from the `site/` directory
4. Assets are synced to S3 with optimized cache headers
5. CloudFront cache is invalidated to ensure fresh content delivery
6. The site is live globally within seconds across 400+ edge locations

This replaces a manual, error-prone process with one that is automated, version-controlled, and production-appropriate.

---

## AWS Well-Architected Alignment

| Pillar | Implementation |
|--------|---------------|
| **Security** | S3 bucket is private with no public access. CloudFront OAC restricts origin access. GitHub Actions uses OIDC federation — zero static AWS credentials. IAM deploy role is scoped to S3 sync and CloudFront invalidation only. HTTPS enforced via viewer protocol redirect. |
| **Reliability** | CloudFront provides built-in failover across 400+ global edge locations. S3 delivers 99.999999999% durability with versioning enabled. SPA error handling returns `index.html` for 403/404 responses. |
| **Performance Efficiency** | CloudFront edge caching with optimized cache policy. HTTP/2 and HTTP/3 enabled. Gzip compression on all responses. Static assets receive 1-year cache headers; HTML receives 60-second headers for fast updates. |
| **Cost Optimization** | No servers to run. S3 + CloudFront free tier covers most small sites. Estimated cost: $1-3/month. `PriceClass_100` limits distribution to US, Canada, and Europe for cost control. |
| **Operational Excellence** | Infrastructure defined as Terraform code. CI/CD pipeline automates build, deploy, and cache invalidation. Deployment summaries posted to GitHub Actions. Rollback via S3 versioning or git revert. |

---

## Repository Structure

```
aws-static-site-pipeline/
├── terraform/
│   ├── main.tf                    # S3, CloudFront, OAC, OIDC, IAM
│   ├── variables.tf               # Input variables
│   ├── outputs.tf                 # Site URL, bucket, role ARN
│   └── terraform.tfvars.example   # Example configuration
├── site/
│   └── index.html                 # Static site (replace with your build)
├── docs/
│   └── architecture.svg           # Architecture diagram
├── .github/
│   └── workflows/
│       └── deploy.yml             # CI/CD: build → S3 sync → CDN invalidation
├── deploy.sh                      # CloudShell one-command deployment
├── .gitignore
└── README.md
```

---

## CI/CD Pipeline

```
Push to main (site/ changes)
        │
        ▼
  ┌───────────┐
  │   Build    │  Package static assets
  └─────┬─────┘
        │
        ▼
  ┌───────────┐
  │  Deploy    │  OIDC auth → S3 sync → CloudFront invalidation
  └─────┬─────┘
        │
        ▼
  Site is live globally
```

The pipeline has two stages:

- **Build** — packages `site/` contents as a deployable artifact (runs on all pushes and PRs)
- **Deploy** — authenticates via OIDC, syncs to S3 with smart cache headers, invalidates CloudFront (runs on `main` only)

Static assets (JS, CSS, images) receive a 1-year `Cache-Control` header for maximum CDN efficiency. HTML files receive a 60-second header so content updates propagate quickly without a full invalidation delay.

---

## Production Design Considerations

- **No public S3 access** — the bucket blocks all public ACLs and policies. Only CloudFront can read from it via Origin Access Control.
- **No static credentials** — GitHub Actions authenticates using OpenID Connect. The IAM role trust policy is scoped to a specific repository. There are no access keys to rotate, leak, or manage.
- **Scoped IAM permissions** — the deploy role can only write to the site bucket and create CloudFront invalidations. Nothing else.
- **SPA routing support** — CloudFront returns `index.html` for 403 and 404 errors, enabling client-side routing frameworks like React Router and Vue Router.
- **Versioned storage** — S3 versioning is enabled, allowing recovery of any previously deployed file version.

---

## Prerequisites

- AWS account with appropriate permissions
- Terraform >= 1.5.0
- GitHub repository
- AWS CLI v2 (included in CloudShell)

---

## How to Deploy

### Option 1: CloudShell (recommended)

```bash
# Upload aws-static-site-pipeline.zip via Actions → Upload file
bash aws-static-site-pipeline/deploy.sh
```

The script handles everything: Terraform init, plan, apply, initial S3 upload, and cache invalidation. It prints the site URL and the three GitHub secrets you need.

### Option 2: Manual

```bash
cd terraform
cp terraform.tfvars.example terraform.tfvars
# Edit terraform.tfvars with your values
terraform init
terraform plan
terraform apply

# Upload site
aws s3 sync ../site/ s3://$(terraform output -raw s3_bucket_name) --delete
```

### After deploy — connect to GitHub

1. Create a GitHub repo and push this code
2. Add three secrets to the repo (Settings → Secrets → Actions):

| Secret | Value |
|--------|-------|
| `AWS_ROLE_ARN` | IAM role ARN from Terraform output |
| `S3_BUCKET` | S3 bucket name from Terraform output |
| `DISTRIBUTION_ID` | CloudFront distribution ID from Terraform output |

3. Push a change to `site/` — the pipeline deploys automatically

---

## Terraform Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `aws_region` | AWS region | `us-east-1` |
| `project_name` | Resource naming prefix | `static-site-pipeline` |
| `environment` | Environment label | `prod` |
| `github_repo` | GitHub repo for OIDC trust (`owner/repo`) | `""` |
| `create_oidc_provider` | Create OIDC provider (false if it already exists) | `false` |
| `cloudfront_price_class` | CDN price class | `PriceClass_100` |

---

## Rollback Strategy

Two approaches depending on the situation:

**Code-level rollback** — revert the commit and push. The pipeline redeploys the previous version automatically.

```bash
git revert HEAD
git push
```

**Object-level rollback** — S3 versioning allows restoring any individual file to a previous version without redeploying.

```bash
aws s3api list-object-versions --bucket <bucket> --prefix index.html
aws s3api get-object --bucket <bucket> --key index.html --version-id <id> index.html
aws s3 cp index.html s3://<bucket>/index.html
```

---

## Security Notes

- The S3 bucket has no public access. All four public access block settings are enabled.
- CloudFront uses Origin Access Control (OAC), which is the current AWS-recommended approach. The older Origin Access Identity (OAI) method is not used.
- GitHub Actions authenticates via OIDC. The IAM trust policy restricts access to a specific GitHub repository. No static AWS credentials exist in this workflow.
- The deploy IAM role follows least privilege: it can only put/get/delete objects in the site bucket and create CloudFront invalidations.
- HTTPS is enforced. HTTP requests are redirected to HTTPS via CloudFront viewer protocol policy.

---

## Business Outcome

The solution achieved what the customer needed:

- **Deployments went from manual to fully automated** — push to `main` and the site updates
- **Global performance improved immediately** — CloudFront edge caching across 400+ locations
- **Operational overhead dropped significantly** — no servers to patch, monitor, or maintain
- **Security posture improved** — no public S3 access, no static credentials, HTTPS enforced
- **Infrastructure became reproducible** — Terraform manages everything as code
- **Monthly cost dropped to $1-3** — from a server-based hosting model to managed services

---

## Future Enhancements

- Custom domain with ACM certificate and Route 53
- CloudFront Functions for header manipulation or A/B testing
- AWS WAF integration for rate limiting and bot protection
- Multi-environment support (staging + production)
- Lighthouse CI integration for automated performance scoring
- Slack/Teams notification on deploy success or failure

---

## Author Perspective

I built this as a cloud architect and consultant. The goal was not to deploy a website — it was to design a **delivery platform** that could be explained to a customer, handed off to an operations team, and maintained without tribal knowledge.

Every decision in this project traces back to a requirement, not a preference. That is how production infrastructure should be built.

---

## License

MIT
