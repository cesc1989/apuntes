# Flujos del Cliente en Salesforce y Ontraport

## Flujo General

OrderlyMeds — General Application Flow

```                                                                              
                                                                                                                                 
  ┌──────────────────────────────────────────────────────────────┐                                                                 
  │                         PATIENT                              │                                                                 
  │                  (orderlymeds.com / mobile app)              │                                                                 
  └─────────────────────┬────────────────────────────────────────┘                                                                 
                        │                                                                                                          
                   Health Screener                                                                                               
                        │
            ┌───────────┴───────────┐
            │                       │
     ┌──────▼──────┐         ┌──────▼──────┐
     │  SALESFORCE │         │  ONTRAPORT  │
     │    FLOW     │         │    FLOW     │
     │   (new)     │         │  (legacy)   │
     └──────┬──────┘         └──────┬──────┘
```

### Salesforce Flow - MemberPeriod States

```
  #Health Screener
        │
        ▼
  #InHealthScreening
        │
        ▼
  #HealthScreeningCompleted
        │
        ▼
  ReadyForCheckin ──────────────────────► CheckinDelayed
        │                                       │
        ▼                                       │
  CheckinCompleted ◄──────────────────────────-─┘
        │
        ▼
  ReadyForProductSelection
        │
        ▼
  ReadyForOrderPayment
        │
        ├──► OrderPaymentSubmitted
        │           │ (Salesforce Flow handles transition)
        │           ▼
        └──► ReadyToCreateVisit
                    │
                    ├──► Referred
                    │
                    ▼
               VisitCreated
                    │
                    ▼
               VisitCompleted
                    │
                    ▼
             PrescriptionWritten
                    │
                    ▼
        WaitingOnPharmacyConfirmation
                    │
                    ▼
           PharmacyOrderConfirmed
                    │
                    ├──► PharmacyOrderShipped
                    │           │
                    │           ▼
                    │    PharmacyOrderDelivered
                    │
                    └──► OrderCanceledByPharmacy
                                 │
                                 └──► #VisitCreated (recreate visit)

  ─── Terminal / exception states ───────────────────────────────
  MissedCheckin
  Canceled
  ImportedFromOntraport
```


### Ontraport Flow - Sales Stage States

```
  #Health Screener
        │
        ▼
  #Prospect
        │
        ▼
  #In HealthScreening
        │
        ▼
  #Health Screening Review
        │
        ▼
  #Consult Requested
        │
        ▼
  #Ready for Payment
        │
        ▼
  #Ready To Submit To Pharmacy
        │
        ▼
  Pharmacy Selected (Ready to create visit/ Visit Created)
        │
        ▼
  Prescription Written (at Pharmacy) (Pharmacy Order Confirmed)
        │
        ▼
  #First Checkin ──────────────────────► Checkin Shipped
        │
        ▼
  Medication Shipped
        │
        ▼
  Ongoing Customer

  ─── Exception states ───────────────────────────────
  #Failed First Purchase
  #Declined Card
  #Subscription Suspended
  #OvershippedMeds
```

### Prescription Services

Beluga Health y Care Validate.

```
  OrderlyMeds (Rails)
          │
          ├──────────────────────────────────────┐
          │                                      │
  ┌───────▼────────┐                  ┌──────────▼──────────┐
  │     BELUGA     │                  │    CARE VALIDATE    │
  │  (Prescription)│                  │      (Cases)        │
  │                │                  │                     │
  │ → Master ID    │                  │ → Case ID (UUID)    │
  │   generated    │                  │   Decision: Approve │
  │   (Visits)     │                  │   or Reject         │
  └───────┬────────┘                  └──────────┬──────────┘
          └──────────────┬───────────────────────┘
                         │
                         ▼
                ┌─────────────────┐
                │    PHARMACY     │
                └────────┬────────┘
                         │
            ┌────────────┼────────────┐
            ▼            ▼            ▼
        PerfectRx    SmartPharma    Casa
            │            │            │
            └────────────┴────────────┘
                         │
                Webhook → OrderlyMeds
                         │
                ┌────────┴────────┐
                │  Order status   │
                │  updated        │
                │  MP state →     │
                │  Shipped /      │
                │  Delivered /    │
                │  Canceled       │
                └─────────────────┘
```

### Patient Portal

Success App

```
  Patient logs in (WorkOS / AuthKit)
          │
          ▼
  Account.salesforce_account_nk present?
          │
     YES  │  NO
          │
     ┌────┴────┐
     ▼         ▼
  Salesforce  Ontraport
  Connector   Connector
          │
          ▼
  Patient Dashboard
    ├── Order status
    ├── Check-in schedule
    ├── Medication history
    └── Chat with provider
```