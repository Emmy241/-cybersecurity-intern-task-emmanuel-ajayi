# Part 2: Deep-Dive Security Analysis  
## Risk Focus: Authorization & Multi-Tenancy Boundaries (IDOR)

## 1. Technical Explanation: How the Attack Works

ClientHub is a multi-tenant SaaS platform where multiple companies access the same application instance while expecting strict logical separation of data. Each request to view or modify customer records includes identifiers such as `company_id` or `customer_id` to scope the data.

In this attack scenario, authorization checks are improperly enforced.

### Step-by-Step Attack Flow

1. A legitimate user (e.g., Support Agent) logs into the web dashboard and accesses a valid customer record belonging to their company.
2. The browser sends a request such as:

`GET /customers/view?company_id=1234&customer_id=5678`

3. The backend validates that the user is authenticated but fails to sufficiently verify that the user is authorized to access `company_id=1234`.
4. The user manually modifies the request parameter:

`GET /customers/view?company_id=1235&customer_id=9012`

5. Because the backend trusts the client-supplied `company_id`, it returns customer data belonging to a different organization.
6. The user gains unauthorized read (and potentially write) access to another tenant’s data.

This vulnerability is classified as an **Insecure Direct Object Reference (IDOR)** caused by missing or incomplete server-side authorization checks.


## 2. Real-World Example

In 2019, **Capital One** suffered a major data breach due to improper access controls and authorization weaknesses in a cloud-hosted environment. An attacker exploited misconfigured permissions to access sensitive customer data stored in AWS S3, affecting over 100 million customers.

Although the technical vectors differ, the underlying issue **failure to enforce strict access boundaries between sensitive data and users** is conceptually similar.

Source:  
https://www.capitalone.com/about/newsroom/capital-one-announces-data-security-incident/


## 3. Defense Strategy

### Control 1: Enforce Server-Side Authorization at the Object Level

**What to implement:**  
Every request that accesses customer or company data must validate ownership on the server side by deriving the tenant context from the authenticated session—not from client-supplied parameters.

**Why it works:**  
This prevents attackers from manipulating identifiers, as access decisions are based on trusted session attributes rather than user input.

**How to verify it works:**  
- Attempt to access resources belonging to another company using modified IDs.
- Confirm the API consistently returns `403 Forbidden`.
- Add automated authorization tests to CI pipelines.

**Trade-off:**  
Increased development effort and slightly more complex backend logic.


### Control 2: Centralized Authorization Middleware or Policy Engine

**What to implement:**  
Introduce a centralized authorization layer (e.g., middleware or policy-based access control) that enforces tenant isolation consistently across web and API endpoints.

**Why it works:**  
Centralization reduces the risk of inconsistent checks and prevents developers from forgetting authorization logic in new endpoints.

**How to verify it works:**  
- Review codebase to confirm all data-access routes pass through the authorization layer.
- Perform negative testing against newly introduced endpoints.

**Trade-off:**  
Initial refactoring effort and potential performance overhead if not designed efficiently.


### Control 3: Security Monitoring and Access Logging

**What to implement:**  
Log all access to sensitive objects with user ID, tenant ID, and request metadata. Create alerts for anomalous access patterns (e.g., sequential access to multiple tenant IDs).

**Why it works:**  
Even if a control fails, logging enables rapid detection and forensic analysis, reducing dwell time.

**How to verify it works:**  
- Simulate unauthorized access attempts and confirm logs are generated.
- Validate alerts trigger when abnormal patterns occur.

**Trade-off:**  
Increased log storage costs and alert tuning requirements.


## 4. Summary

Authorization and multi-tenancy failures represent one of the highest-impact risks for SaaS platforms. Unlike authentication flaws, these vulnerabilities can be exploited by legitimate users with minimal technical skill. Addressing them requires strict server-side enforcement, architectural discipline, and ongoing monitoring.

Failure to remediate this class of vulnerability would significantly undermine ClientHub’s readiness for enterprise customers.
