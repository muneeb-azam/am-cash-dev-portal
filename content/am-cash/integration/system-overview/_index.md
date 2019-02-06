+++
title = "System Overview"
pre = "2. "
weight = 20
+++
---

This section provides an overview of the Instant POS Platform. It discusses the different components that form the platform and the relationships between these components. The integration guide pertains to AM Cash POS API portion of the Instant POS Platform.  
  
## System Components  

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