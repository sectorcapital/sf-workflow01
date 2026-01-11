## Workflow Diagram

```mermaid
flowchart LR
  A["Acquisition__c<br/>Rep Discovery"] -->|Elevate Requested| B["Opportunity__c / Deal__c<br/>Closer Engineering"]

  B --> C{Closer Decision}
  C -->|Reject / Not Ready| A
  C -->|Accept| U[Underwriting]

  U --> D["ExitStrategy__c (child)<br/>Score and Rank Strategies"]
  D --> E[Best Strategy Selected]
  E --> F["Offer__c (child)<br/>Draft -> Sent -> Accepted/Rejected"]
