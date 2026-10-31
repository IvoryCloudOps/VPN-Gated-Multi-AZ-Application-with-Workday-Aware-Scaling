# VPN-Gated Multi-AZ Application with Workday-Aware Scaling🏢📈
Status: Complete✅

Environment status: AWS lab deployed and validated, then intentionally decommissioned to avoid recurring charges

Focus: AWS · Terraform · Linux · Networking · Cloud Operations · Monitoring · CI/CD

Designed and deployed an internal AWS distribution operations platform using private networking, an EC2 Auto Scaling Group, an internal Application Load Balancer, Multi-AZ RDS MySQL, and operational monitoring. The Flask application uses synthetic inventory, order, and shipment data; the engineering focus is the surrounding infrastructure and its reliability.

[Read the full engineering case study on Medium](https://medium.com/@ivorycloudops/building-a-private-multi-az-aws-operations-platform-for-regional-distribution-%EF%B8%8F-8dafcc20e7de?sharedUserId=ivorycloudops) — Architecture decisions, selected Terraform code, deployment evidence, screenshots, and production trade-offs. 

# 1. Business Problem🏢
A simulated regional distributor needed a private, reliable platform for warehouse inventory, orders, shipments, and reporting across multiple facilities. The solution needed network isolation, configurable workday scaling, visibility into unhealthy resources, and database backup capability. This project validates infrastructure behavior, not application user management or a specific throughput target.
# 2. Architecture🏗️

![https://cdn-images-1.medium.com/max/1200/1*R9AtzY91INhs9X6-rqvz9w.png](https://cdn-images-1.medium.com/max/1200/1*XQEQM5QwwXoup8-hH2wxyA.png)

- Private access: AWS Client VPN to an internal ALB; separate public, application, and database subnets across two Availability Zones.
- Application tier: EC2 Auto Scaling Group bootstrapped from S3, with ALB health checks and scheduled/CPU-based scaling.
- Data: Private Multi-AZ RDS MySQL with Secrets Manager-managed credentials; protected S3 object storage.
- Operations: CloudWatch alarms, SNS, Systems Manager, Boto3 audit script, and a database recovery runbook.
- Delivery: Terraform remote state and GitHub Actions OIDC-based format, validation, and plan checks.
# 3. Key Decisions & Why🤔
- Private ALB + Client VPN: Keep an internal business application off the public internet.
- Two AZs with two healthy ALB targets: Reduce reliance on a single application instance or availability zone.
- Scheduled + CPU target-tracking scaling: Address predictable workday demand and unexpected load changes.
- Multi-AZ MySQL + managed secrets: Protect relational business data and avoid credentials in source code.
- CloudWatch and Boto3 checks: Make infrastructure state observable and easier to troubleshoot.
- Single NAT Gateway: Reduce lab costs while documenting the associated resiliency limitation.
Validation highlights: The recorded deployment showed two healthy instances and two healthy ALB targets across separate AZs, an accessible internal application, RDS Multi-AZ enabled, seven-day automated backups, and configured CloudWatch alarms. See Medium for the evidence and limitations.
# 4. What I'd Do Differently at Production Scale📈
- Add one NAT Gateway per AZ and require TLS for internal application traffic.
- Verify failover, load-based scaling, restore procedures, and notification delivery with repeatable tests.
- Enable VPN connection logging and tighten GitHub OIDC trust and IAM scopes.
- Use CI/CD approval gates, production database deletion protection, and stronger retention controls.
- Add application-level authentication and role-based access, integrated with corporate SSO; validate expected concurrency with load tests.
- Expand application metrics, log aggregation, and documented recovery objectives.
Source: [`terraform/`](terraform/), [`application/`](application/), [`scripts/`](scripts/)
Runbook: [`docs/runbooks/rds_backup_recovery.md`](docs/runbooks/rds_backup_recovery.md)
Detailed case study: Read on Medium

- **Automated `terraform apply`** behind a manual approval gate, rather than a plan-only pipeline.

- **Centralized logging** (CloudWatch Logs → a proper aggregation/SIEM tool) instead of per-service log groups, once there's more than one workload to correlate across.

- **Multi-region DR**, if the business risk of a full regional outage justified the added cost and complexity.
