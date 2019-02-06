+++
title = "Integration Environments"
pre = "3. "
weight = 30
+++
---

There are two sets of environments used by partners to integrate with AM Cash POS API: 

* The "Instant POS Platform" Live environment. This environment contains Production quality code with live data.
* The "Instant POS Platform" Sandbox environment. This environment contains Production quality code with sandbox data (not real customer data). 

|Environment            |Domain                          |
|-----------------------|--------------------------------|
|Sandbox API            |https://sandbox.api.loyalty.com |
|Live (Production) API  |https://api.loyalty.com         |  

These environments are configured similarly for consistent production-level quality, whether the partner has deployed to production or is still performing non-production testing. All integration environments must be accessed via HTTPS.  

## AM Cash and Instant POS Platform SLA

* Over 99% of AM Cash balance inquiries and redemption transactions are processed between Partners and AMRP without incident.  

* Requests should take no longer than 7 seconds. On average, requests should take around 500-750ms to complete.  

* On the rare occasion when a Collector attempts to redeem from their account but the Partners POS does not receive a ‘success’ message, they may have miles removed from their account with no corresponding redemption off their purchase transaction.  

* For the majority of these cases there is an automated process that detects and refunds the Cash Miles within 20 minutes in Production environment.  

