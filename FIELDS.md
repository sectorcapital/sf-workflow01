# Field Taxonomy & Object Boundaries (SF Build Spec)

Goal: make every field **obvious** (what it is, who owns it, and when it’s required) so page layouts + automation are buildable and explainable.

---

## Prefix Standards (Naming Convention)

Use a consistent prefix so fields self-identify:

- **PROP_** = property facts / condition
- **MLS_** = listing info + history
- **FC_** = foreclosure timeline / auction
- **SCOOP_** = rep call scripting, questions, tone
- **MOT_** = motivation, urgency, seller situation
- **OCC_** = occupancy / tenant
- **UW_** = underwriting stage gates & controls
- **EQ_** = equity / valuation / repairs / liens / loans
- **EXIT_** = best strategy pointer/summary (on Opportunity)
- **STRAT_** = strategy scoring/assumptions (on ExitStrategy__c)
- **OFFER_** = offer terms (on Offer__c)
- **NS_** = net sheet (profit model / costs / proceeds)
- **CF_** = cashflow model (rent/hold/wrap)
- **OPS_** = routing, ownership, dates, SLAs, automation flags

---

## Object Boundaries (Where Fields Live)

### A) Acquisition__c (Rep Discovery Hub)
**Owner:** Rep  
**Purpose:** discovery + rapport + motivation triage + follow-up scheduling  
**Do NOT put offers or strategy scoring here.**

**Core Sections**
1) **Property Info**
- PROP_Address
- PROP_Beds / PROP_Baths
- PROP_Sqft / PROP_LotSqft
- PROP_YearBuilt
- PROP_Pool / PROP_Garage / PROP_Parking
- PROP_Condition (picklist)
- PROP_Situation (notes)

2) **Scoop Script (Rep Prepared Questions)**
- SCOOP_QuestionPlan (long text) — “questions I plan to ask”
- SCOOP_VoicemailLeft (checkbox)
- SCOOP_AgentTone (picklist: friendly / guarded / hostile / unknown)
- SCOOP_CallSummary (long text) — what we learned without underwriting

3) **Motivation / Urgency (Triage)**
- MOT_Level (picklist 1–5)
- MOT_UrgencyLevel (picklist 1–5)
- MOT_Summary (long text) — rep’s plain-English takeaway

4) **Occupancy**
- OCC_Status (owner / tenant / vacant / unknown)
- OCC_TenantNotes (text)

5) **Foreclosure**
- FC_AuctionStage (picklist)
- FC_AuctionDate (date)
- FC_OriginalAuctionDate (date)
- FC_DateUnknownReason (text) — only if date missing

6) **MLS Listing Info**
- MLS_ListPrice (currency)
- MLS_OriginalListPrice (currency)
- MLS_LastChangeDate (date)
- MLS_PriceDropHistory (long text or structured later)

7) **Rep Workflow Controls**
- OPS_Stage (picklist)
- OPS_NextFollowUpDate (date)
- OPS_ElevateRequested (checkbox)
- OPS_ElevationNotes (long text)

**Elevation Output (handoff snapshot to Opportunity)**
- OPS_ElevateRequested = TRUE triggers Opportunity creation (see TRIGGERS.md)

---

### B) Opportunity__c / Deal__c (Closer Engineering Hub)
**Owner:** Closer (until Offer Writing)  
**Purpose:** underwriting + decisioning + selecting best strategy  
**Contains snapshot of rep intel, NOT the full scoop script.**

**Key Sections**
1) **Rep Intel Snapshot**
- MOT_Summary (copied)
- OPS_ElevationNotes (copied)
- MOT_Level / MOT_UrgencyLevel (copied)
- FC_AuctionDate / FC_AuctionStage (copied)
- OCC_Status (copied)

2) **Closer Decision**
- UW_Decision (picklist: Accept / Reject / Return to Rep)
- UW_RejectReason (picklist or text)
- OPS_ReturnToRepNotes (long text)
- OPS_NextFollowUpDate (date)

3) **Underwriting (Numbers)**
- EQ_ARV (currency)
- EQ_Repairs (currency)
- EQ_LienSummary (long text) OR EQ_LienCount + EQ_TotalLiensEstimate
- EQ_LoanOriginationDate (date) — first loan origination (as you flagged)

4) **Stage Gates**
- UW_UnderwritingComplete (checkbox)
- EXIT_HasCandidateStrategy (checkbox or rollup)
- EXIT_BestStrategy (lookup to ExitStrategy__c) OR EXIT_BestStrategyType (picklist)

---

### C) ExitStrategy__c (Child of Opportunity__c)
**Owner:** Closer  
**Purpose:** multiple possible strategies, scored and ranked  
**1 record per strategy** (avoid multi-select fields).

**Core Fields**
- STRAT_Type (picklist)
  - Cash Flip
  - Sub2 Flip
  - Sub2 Hold (rent)
  - Sub2 Retail (end buyer)
  - Novation
  - Equity Share / Partner
  - Assignment (Wholesale)
  - Facilitation Fee
  - Equity Shield Listing
  - Buy Sub2 -> Wrap / Owner Finance
  - Agent Rep Buyer (buy-side agent)

- STRAT_Candidate (checkbox)
- STRAT_PriorityRank (number)
- STRAT_Confidence (picklist 1–5)
- STRAT_Notes (long text)

**Financial Modeling (optional v1, expandable later)**
- NS_EstNetProfit (currency)
- NS_EstROI (percent)
- CF_EstMonthlyCashflow (currency)

---

### D) Offer__c (Child of Opportunity__c)
**Owner:** Offer Writer  
**Purpose:** track one or more offers (draft/sent/counter/accepted/rejected)

**Core Fields**
1) **Offer Identity**
- OFFER_Status (picklist: Draft / Sent / Countered / Accepted / Rejected / Withdrawn / Expired)
- OFFER_Type (picklist: Cash / Sub2 / Novation / EquityShare / FacilitationFee / AgentRepBuyer / Other)
- OFFER_AB2424_Postponement (checkbox)

2) **Who is the Buyer / Who Writes**
- OFFER_BuyerType (picklist: InHouse / Investor / Retail / ListingAgentWrites / ListingAgentWritesForOurBuyer)
- OFFER_Buyer (lookup Account/Contact)
- OFFER_EMD (currency)

3) **Offer Terms**
- OFFER_PurchasePrice (currency)
- OFFER_CloseDateTarget (date)
- OFFER_OwnerCarry (checkbox)

4) **Sub2 Loan Details (if applicable)**
- OFFER_1stLoanBalance (currency)
- OFFER_2ndLoanBalance (currency)
- OFFER_OtherLiensEstimate (currency)

---

## “Net Sheet” Guidance (Where It Belongs)
- **Strategy-level numbers** belong on **ExitStrategy__c** (because each strategy has different economics).
- If you want a “single headline net sheet,” roll up from the **Best Strategy** to Opportunity (later).

---

## Build Notes (Practical)
- Page layouts should mirror ownership:
  - Reps mainly see **Acquisition**
  - Closers mainly see **Opportunity + ExitStrategies**
  - Offer Writers mainly see **Offer**
- Validation rules should be stage-gated (required-by-stage, not globally required).

