# TechCorp IAM Integration Challenges & Mitigations
## Legacy, Cloud, and SaaS Integration Obstacles – Assessed & Addressed

---

## Executive Summary

TechCorp operates a heterogeneous environment spanning:
- **Legacy systems:** Mainframe, on-prem proprietary apps
- **SaaS sprawl:** 200+ sanctioned + unknown shadow IT applications
- **Multi-cloud:** AWS, Azure, GCP with inconsistent IAM models
- **Hybrid identity:** On-prem AD + cloud directories

This matrix documents each integration challenge and the specific mitigation strategy employed.

---

## Challenge Matrix

### 🏛️ Legacy Systems

| Challenge | Description | Risk Level | Mitigation Approach | Solution |
|----------|-------------|-----------|---------------------|----------|
| **Mainframe authentication** | No support for SAML/OIDC; requires RACF/ACF2 | High | Deploy PAM connector with credential injection | CyberArk PAM for mainframe; session recording |
| **On-prem custom apps** | Built with embedded authentication; no federation | Medium | Application-level proxy or agent | Okta LDAP agent / Entra ID Application Proxy |
| **No SCIM support** | Cannot automate provisioning/deprovisioning | High | Custom connector development | IGA vendor professional services; REST APIs where available |
| **Directory fragmentation** | Multiple disconnected LDAP directories | Medium | Consolidate or virtualize | Virtual directory service; sync to cloud identity store |

**Decision:** Maintain PAM-based access for mainframe; modernize top 20 legacy apps via federation proxies by Phase 2.

---

### ☁️ SaaS Sprawl & Shadow IT

| Challenge | Description | Risk Level | Mitigation Approach | Solution |
|----------|-------------|-----------|---------------------|----------|
| **Unsanctioned SaaS apps** | Users provisioning identity outside IT control | High | Discover via CASB/SSO logs | Deploy CASB tool; block high-risk apps |
| **Inconsistent SSO adoption** | Some SaaS apps use social login, not corporate IdP | Medium | Phased onboarding, business case | Prioritize by criticality; SSO mandate for new contracts |
| **SCIM availability variance** | Some apps support SCIM, others require custom | Medium | SCIM where available; custom where required | Standardize on SCIM 2.0; maintain connector library |
| **Just-in-time provisioning gaps** | JIT creates accounts but doesn't deprovision | Medium | Complement with periodic reconciliation | IGA-driven reconciliation sweeps |

**Decision:** SSO mandate for all sanctioned SaaS by Phase 2; JIT + deprovisioning reconciliation by Phase 3.

---

### 🌐 Multi-Cloud IAM

| Challenge | Description | Risk Level | Mitigation Approach | Solution |
|----------|-------------|-----------|---------------------|----------|
| **Inconsistent policy models** | AWS IAM ≠ Azure AD ≠ GCP Cloud Identity | High | Abstraction layer; central policy definitions | Cloud IAM policy-as-code; Terraform modules |
| **Entitlement visibility gap** | No single view of who has what across clouds | Critical | Deploy CIEM tool | Entra ID Permissions Management / AWS IAM Identity Center |
| **Overprivileged service accounts** | Machine identities with excessive permissions | High | Service account governance | PAM for secrets; regular certification |
| **Cloud-native IAM bypass** | Developers create local IAM roles outside governance | High | Guardrails + self-service | Cloud IAM templates with approval workflows |

**Decision:** Deploy CIEM in Phase 2; enforce cloud IAM policy-as-code by Phase 3.

---

### 🔄 Hybrid Identity Sync

| Challenge | Description | Risk Level | Mitigation Approach | Solution |
|----------|-------------|-----------|---------------------|----------|
| **Double-hop latency** | On-prem AD → cloud → app | Medium | Direct sync with failover | Entra ID Connect / Okta LDAP agent; health monitoring |
| **Password hash sync security** | Risk of on-prem credential exposure | Medium | Enable PHS with conditional access policies | Use pass-through authentication where feasible |
| **Object lifecycle mismatch** | Deleted on-prem objects not synced promptly | Medium | Soft-delete + cleanup job | IGA-driven deprovisioning, not directory sync alone |
| **Multi-forest complexity** | Multiple AD forests with same users | Medium | Forest trust + source of truth alignment | Designate authoritative forest; sync to cloud once |

**Decision:** Maintain hybrid sync with failover; deprovisioning driven by IGA, not directory sync.

---

### 👤 User Experience vs. Security

| Challenge | Description | Risk Level | Mitigation Approach | Solution |
|----------|-------------|-----------|---------------------|----------|
| **MFA fatigue** | Users ignore or approve fraudulent pushes | High | Move to phishing-resistant MFA | FIDO2/WebAuthn; number matching |
| **Passwordless adoption barriers** | Hardware token cost; user training | Medium | Tiered rollout | Executives/IT first; broad pilot in Phase 2 |
| **Access request delays** | Manual approval chains | Medium | Automated workflows + AI triage | Agentic AI for low-risk auto-approval |
| **Multiple credentials** | Users maintain separate passwords for different systems | High | SSO-first architecture | 90%+ SSO coverage target |

**Decision:** Passwordless for all executives by Phase 2; enterprise-wide by end of Year 2.

---

## Integration Architecture Principles

1. **API-first:** Prefer REST/SCIM over custom agents
2. **Standardize protocols:** SAML for SSO, SCIM for provisioning
3. **Minimize custom code:** Use vendor connectors; build only when necessary
4. **Fail securely:** Deny by default; log all integration failures
5. **Monitor everything:** Integration health dashboards for all connectors

---

## Integration Health Dashboard (Proposed)

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
CONNECTOR HEALTH STATUS ── PHASE 1 PILOT
┌────────────────┬──────────┬─────────┬──────────────┐
│ Connector │ Status │ Latency │ Last Success │
├────────────────┼──────────┼─────────┼──────────────┤
│ Workday → IGA │ ✅ Healthy│ 3.2s │ 09:47:23 │
│ IGA → Entra ID │ ✅ Healthy│ 1.8s │ 09:47:25 │
│ Entra ID → SFDC│ ✅ Healthy│ 2.1s │ 09:47:28 │
│ PAM → AWS │ ⚠️ Degraded│ 8.9s │ 09:45:12 │
│ Legacy App Proxy│ ✅ Healthy│ 4.5s │ 09:47:01 │
└────────────────┴──────────┴─────────┴──────────────┘
│                                                         │
└─────────────────────────────────────────────────────────┘
```
