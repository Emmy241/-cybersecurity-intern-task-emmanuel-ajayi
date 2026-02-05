# Part 3: Incident Response Plan

## Incident: Cross-Tenant Data Access via URL Parameter Manipulation - IDOR

### Incident Summary
A support agent reported being able to view customer records belonging to another company by modifying a `company_id` parameter in the web dashboard URL. This behavior indicates a potential data exposure incident affecting tenant isolation.


## 1. Immediate Actions (First 1 Hour)

1. Acknowledge the report and classify the incident as high severity.
2. Temporarily restrict or disable affected endpoints where feasible.
3. Preserve application, API, and database logs to maintain evidence integrity.
4. Notify engineering, security, and leadership teams.
5. Monitor for additional suspicious access patterns.
   

## 2. Investigation Checklist

### Scope Assessment
- Identify all endpoints that accept tenant-related identifiers.
- Determine whether the issue affects the web application, mobile application, or APIs.

### Data Exposure
- Identify the types of data accessible through the vulnerability.
- Determine the number of affected companies and customer records.

### Evidence of Prior Exploitation
- Review logs for repeated or sequential access to multiple tenant identifiers.
- Correlate user IDs, IP addresses, and timestamps.
- Assess whether access patterns indicate manual or automated exploitation.


## 3. Root Cause Analysis

### Possible Technical Causes
1. Missing server-side object-level authorization checks.
2. Reliance on client-side logic for access control enforcement.
3. Inconsistent authorization implementation across endpoints.

### Most Likely Cause
Missing server-side object-level authorization checks, where the backend fails to verify that requested resources belong to the authenticated user’s tenant.


## 4. Fix Validation

1. Attempt to access resources using modified tenant identifiers.
2. Confirm unauthorized access is denied with `403 Forbidden`.
3. Validate enforcement across all user roles.
4. Test API endpoints independently of the web interface.
5. Verify logging and alerting for failed authorization attempts.
6. Review code to confirm centralized authorization enforcement.


## Conclusion

This incident highlights the importance of strict server-side authorization enforcement in multi-tenant platforms. A structured response, thorough investigation, and disciplined validation process are essential to prevent recurrence and maintain customer trust.
