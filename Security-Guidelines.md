# Security Guidelines for Self-Hosted GitHub Runners

## Introduction

Self-hosted runners provide organizations with flexibility to customize the environment for GitHub Actions workflows. However, they introduce significant security risks if not properly configured and managed. This document outlines security requirements and hardening guidelines for self-hosted runners based on industry best practices and recent supply chain attacks.

## Core Security Principle

> **ALWAYS ASSUME SELF-HOSTED RUNNERS WILL EXECUTE ARBITRARY CODE**

The foundation of all security controls for self-hosted runners must be built on this critical assumption: **any self-hosted runner can and will execute arbitrary code from untrusted sources**. This is not a risk to mitigate but a fundamental characteristic to design around.

All security architecture decisions should be made working backward from this assumption:
- Runners will execute malicious code
- Runners will be compromised
- Attackers will attempt to persist and pivot from runners
- Sensitive credentials on runners will be stolen

The security controls outlined in this document are designed to contain damage when (not if) a runner executes malicious code.

## Understanding the Risks

Self-hosted runners execute code from GitHub repositories, including potentially untrusted code from pull requests. Unlike GitHub-hosted runners which use ephemeral VMs that are destroyed after each job, self-hosted runners can be compromised persistently if not properly secured.

### Notable Attack: 

TBD

## Secure Self-Hosted Runner Architecture

A secure self-hosted runner architecture should be designed based on the core principle that runners will execute arbitrary code. The architecture must contain and limit the damage from inevitable compromise.

### Key Components

#### 1. Organizational Boundary
Clear separation between GitHub and internal environment, with well-defined entry points and protective measures at the boundary.

#### 2. Runner Controller/Orchestrator
A central system that manages the lifecycle of ephemeral runners, including:
- Creating fresh runner instances for each job
- Ensuring runners are properly isolated
- Destroying runners immediately after job completion
- Enforcing security policies consistently

#### 3. Ephemeral Runner Pool
- Isolated runners (containers or VMs) that are created on-demand
- Each runner exists only for the duration of a single job
- No state persists between different job executions
- Runners have no ability to modify their host environment

#### 4. Restricted Network
- Limited network access for runners
- Egress filtering to prevent unauthorized connections
- Segmentation between runners and internal systems
- No direct access to sensitive internal services

#### 5. Secure Secret Management
- Just-in-time access to credentials
- Minimal privileges for each credential
- Automatic rotation of secrets
- Audit logging of all secret access

### Architecture Flow

1. GitHub sends webhook events for workflow triggers to a dedicated webhook receiver
2. The webhook receiver validates the request and forwards it to the runner controller
3. The runner controller provisions a new, isolated, ephemeral runner instance
4. The runner executes the workflow in this isolated environment
5. If secrets are needed, they are provided through a secure just-in-time mechanism
6. Upon job completion (success or failure), the runner instance is immediately destroyed
7. All logs are forwarded to a central monitoring system for analysis

## Security Requirements

### 1. Workflow Approval Controls

- **CRITICAL**: Configure "Require approval for all outside collaborators" rather than the default "Require approval for first-time contributors"
- Implement additional review for any workflows that request access to secrets
- Limit self-hosted runners to internal workflows and trusted contributors only

### 2. Runner Isolation and Ephemerality 

- Use ephemeral runners that are created fresh for each job and destroyed afterward
- Implement isolated environments (containers, VMs) for each job execution
- Never reuse runner environments between workflow runs
- Use read-only filesystems where possible to prevent persistent modifications

### 3. Access Control

- Restrict which repositories can use self-hosted runners (use runner groups)
- Limit the repositories that each runner group can access
- Use dedicated runners for specific sensitive workflows only
- Configure runner groups at the organization level with granular access policies
- Disable self-hosted runners for public repositories unless absolutely necessary

### 4. Secret Management

- Never store sensitive credentials on self-hosted runners
- Use GitHub's OIDC (OpenID Connect) for authentication to cloud services
- Implement just-in-time credential access instead of persistent credentials
- Monitor for and rotate compromised credentials immediately
- Use short-lived access tokens with minimal privileges

### 5. Network Security

- Implement network isolation for runners
- Restrict outbound internet access to only necessary services
- Block unnecessary incoming connections
- Use private networks for communication between runners and internal services
- Consider using a dedicated VLAN or subnet for runner traffic

### 6. Monitoring and Logging

- Implement comprehensive logging of all runner activities
- Set up alerts for suspicious activities like unauthorized workflow runs
- Monitor for unusual network traffic patterns or credential usage
- Maintain audit trails for all actions performed by workflows
- Regularly review logs for signs of compromise

## Hardening Guidelines for Self-Hosted Runners

### Runner Host Configuration

- Use a minimal, hardened OS with only required services
- Apply security patches promptly (automated when possible)
- Use the latest version of the GitHub Actions runner software
- Implement endpoint protection and EDR solutions
- Remove unnecessary software, services, and user accounts

## Implementation Checklist

Use this checklist to ensure you've addressed key security controls:

- [ ] Configure "Require approval for **all** outside collaborators"
- [ ] Implement ephemeral runners that are destroyed after each job
- [ ] Use isolated environments (containers or VMs) for each job
- [ ] Restrict which repositories can use self-hosted runners
- [ ] Implement just-in-time credential access instead of persistent credentials
- [ ] Configure network isolation for runners
- [ ] Set up monitoring and alerting for suspicious activities
- [ ] Maintain audit logs of all runner activities
- [ ] Regularly update runner software and host operating systems
- [ ] Have an incident response plan for compromised runners

## Conclusion

Self-hosted runners provide powerful capabilities but require careful security configuration to prevent them from becoming an attack vector. By implementing the security requirements and hardening guidelines in this document, organizations can significantly reduce the risk of supply chain attacks through their CI/CD pipeline.

**Remember the core principle**: Always assume runners will execute arbitrary code and design your security architecture accordingly. This mindset shift from "preventing compromise" to "containing damage from inevitable compromise" is essential for securing self-hosted runners.

Regular security reviews and updates to these controls are essential as attack techniques continue to evolve.
