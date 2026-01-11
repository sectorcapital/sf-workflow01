# Trigger Matrix (Salesforce Automation Spec)

This document defines the **stage gates**, **routing**, and **automation triggers** for the Rep → Closer → Offer Writer workflow.

---

## Objects
- **Acquisition__c** = Rep Discovery + motivation triage
- **Opportunity__c (Deal__c)** = Closer underwriting + engineering (created on elevate)
- **ExitStrategy__c** (child of Opportunity) = one record per strategy, scored + ranked
- **Offer__c** (child of Opportunity) = one record per offer (draft/sent/counter/accepted)

---

## Acquisition__c Stages (Rep)
Suggested stages:
- Discovery
- Nurture / Follow-up
- Elevated
- Dead / Do Not Pursue

---

## Opportunity__c Stages (Closer/Offer Writer)
Suggested stages:
- Review
- Underwriting
- Strategy Selected
- Offer Writing
- Offer Out
- Closed Won / Closed Lost
- Returned to Rep

---

## Trigger Matrix

| Trigger / Event | Object | Required Inputs (Gate) | Automation (System Actions) | Owner / Routing | Stage Change |
|---|---|---|---|---|---|
| Rep requests elevation | Acquisition__c | MOT_Level, MOT_UrgencyLevel, ElevationNotes, FC_AuctionDate (or FC_DateUnknownReason) | Create Opportunity__c; link to Acquisition; copy snapshot fields | Assign Opportunity to **Closer Queue/User** | Acquisition → Elevated; Opportunity → Review |
| Closer rejects / not ready | Opportunity__c | RejectReason, NextFollowUpDate | Update linked Acquisition with reject note + follow-up date; notify Rep | Assign Acquisition back to Rep | Opportunity → Returned to Rep; Acquisition → Nurture |
| Closer accepts for underwriting | Opportunity__c | — | (Optional) Create default ExitStrategy__c records | Closer | Opportunity → Underwriting |
| Underwriting marked complete | Opportunity__c | EQ_ARV, EQ_Repairs, EQ_LienSummary, LoanOriginationDate (if required) | Validate completeness | Closer | stays Underwriting until Best Strategy selected |
| Strategy marked Candidate = TRUE | ExitStrategy__c | StrategyType | Roll up “HasCandidateStrategy” to Opportunity (or calculate) | Closer | — |
| Best strategy selected | Opportunity__c | EXIT_BestStrategy (lookup) OR BestStrategyType | Create Offer__c Draft; prefill OfferType from BestStrategy | Assign Offer__c to **Offer Writer Queue/User** | Opportunity → Offer Writing |
| Offer sent | Offer__c | Buyer, OfferPrice, EMD (if applicable), OfferType | Update Opportunity offer summary fields | Offer Writer | Opportunity → Offer Out; Offer__c status → Sent |
| Offer accepted | Offer__c | AcceptedDate | Update Opportunity outcome; lock edits if desired | Ops/Closer | Opportunity → Closed Won |
| Offer rejected/expired | Offer__c | OutcomeReason | Option: create new Offer draft or return to Strategy Selected | Offer Writer/Closer | Opportunity → Strategy Selected (or stay Offer Out) |

