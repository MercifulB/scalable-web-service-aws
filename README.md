# Scalable Web Service on AWS

## Architecture

```text
User
  ↓
Application Load Balancer (ALB)
  ↓
ECS Fargate Service
  ↓
Docker Containers across multiple Availability Zones
  ↓
CloudWatch Logs & Metrics
```

This project deploys a containerized web service using AWS ECS Fargate behind an Application Load Balancer. Infrastructure is provisioned entirely using Terraform.

## Scaling

The ECS service horizontally scales between 2 and 6 Fargate tasks using ECS Service Auto Scaling.

### Scaling Trigger
Scaling is triggered by:

- ECS average CPU utilization
- Target tracking policy
- Target CPU threshold: 20% (adjusted lower for demo purposes)

During load testing with `hey`, the service successfully scaled from 2 tasks to 6 tasks automatically.

When traffic drops, ECS scales the service back down automatically.

## High Availability

The service runs across multiple Availability Zones and starts with 2 running tasks to survive the loss of a single task or compute instance without downtime.

The ALB performs health checks against `/health` and routes traffic only to healthy targets.

## Observability

CloudWatch was used for:

- ECS CPU and memory metrics
- Task scaling visibility
- ALB request metrics
- Container logs

## Security / IAM

- IAM task execution roles are used instead of long-lived credentials
- Security groups restrict traffic between the ALB and ECS tasks
- TLS termination is handled at the ALB using ACM certificates

## What I Would Add With Another Week

- Private subnets and NAT Gateway architecture
- CI/CD pipeline using GitHub Actions
- Prometheus + Grafana dashboards
- AWS WAF for rate limiting and security protection
- Request-count-based scaling instead of CPU-only scaling
- Blue/green deployments

## One Thing I Cut For Time

I cut a full private subnet + NAT Gateway networking setup to reduce implementation time and cost. For this assignment, the simpler public-subnet Fargate architecture still demonstrates scaling, resiliency, observability, and infrastructure-as-code effectively.

## Notes

## Notes

The service is publicly accessible through the custom domain `merciful.work`.

HTTPS infrastructure and ACM certificate integration are fully configured in Terraform. At submission time, ACM DNS validation was still propagating through Namecheap DNS records. Once the certificate status changes to `Issued`, running `terraform apply` completes ALB HTTPS listener creation and HTTP to HTTPS redirection automatically.
