+++
title = "Introduction"
pre = "1. "
weight = 10
+++
---

## Purpose of Documentation
The purpose of this document is to provide a guide on how to integrate with the AM Cash POS API. As part of this exercise this document provides information on RESTful HTTP requests and responses, error codes, reconciliation process, integration environments, and contact information.

### Target Audience
This document is intended to be used by LoyaltyOne Co.  partners’ IT departments that want to integrate with the AM Cash POS API.

### Terms & Acronyms

**Instant Program**
: A LoyaltyOne program that enables Air Miles Collectors to check their balances and make on-the-spot “instant” redemptions at the Partner’s stores.   
  
**Instant POS Platform**
: Refers to a software/hardware component that handles messages to provide balance and other information, and to process redemptions and reversals (voids).  
  
**AM Cash POS API**
: A RESTful extension of the Instant POS Platform that simplifies legacy ISO 8583 messages into HTTP request/response paradigm. This extension is the subject of this Integration guide.  
  
**Partner Platform**
: Sometimes called Acquirer, is a software/hardware component that sends messages to the AM Cash POS API (Instant POS Platform).  
  
**AMRM**
: AIR MILES Rewards Miles, is the loyalty program currency that Collectors can earn and redeem for travel, merchandise and card rewards.

## System Components  

This section provides an overview of the Instant POS Platform. It discusses the different components that form the platform and the relationships between these components. The integration guide pertains to AM Cash POS API portion of the Instant POS Platform.  

The following figure shows the Instant POS Platform components:
<object align="left" style="margin-right:50%"> 
![POS Platform Overview](/images/pos-platform.png)
</object>

The main components of the Instant POS Platform are the following: 

* Partner Platform 
* AM Cash API (AWS portion of the Instant POS Platform)
* Instant POS Platform  

The Partner Platform receives transaction information (e.g. balance and AM Cash Inquiry requests) from POS terminals, and then sends the appropriate HTTP requests Instant POS Platform’s AM Cash API for processing. The POS terminals are not shown in this figure; how they communicate with the Partner Platform is managed by the partner. Upon receiving the messages, the Instant POS Platform communicates with internal databases and other systems to process the request message. Once the message is processed, an appropriate response message is sent to the Partner Platform.  

The Instant POS Reconciliation is a software component that is used to support the reconciliation process (see [6. Reconciliation Process](../reconciliation)). It accepts and sends reconciliation files via SFTP on a scheduled basis.
