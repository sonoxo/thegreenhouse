# Born2BGreat Behavioral Health Ontology

**Namespace:** `urn:born2bgreat:greenhouse:behavioral-health:v1:`  
**Owner domain:** `born2bgreat.org`  
**Green House module:** `B2G-BH-1.0`

This directory defines the machine-readable ontology, privacy labels, access-policy vocabulary, and **synthetic-only** example instances for Born2BGreat behavioral-health workflows.

## Critical data boundary

The `sonoxo/thegreenhouse` repository is public. **Never commit real PHI/ePHI, substance-use-disorder patient records, Medicaid identifiers, names, dates of birth, addresses, clinical notes, referral packets, signatures, or other patient-identifiable records to this repository.**

Production instances belong in a separately provisioned protected clinical datastore with contractual, administrative, physical, and technical safeguards appropriate to HIPAA and, when applicable, 42 CFR Part 2. This repository contains schema only.

```text
PUBLIC GREEN HOUSE REPOSITORY
ontology + policy vocabulary + synthetic examples
                |
                | schema/version reference only
                v
GREENHOUSE::B2G_SECURE_CLINICAL_VAULT
identity vault + clinical graph + document store + immutable audit trail
                |
                +--> de-identification pipeline --> analytics/research surface
```

## Design principles

1. **Separate identity from clinical content.** Direct identifiers live in `PatientIdentity`; clinical objects reference an opaque `patientKey`.
2. **Attribute-based authorization.** Access decisions combine workforce role, care-team relationship, purpose of use, data sensitivity, consent/authorization, and emergency/break-glass state.
3. **Minimum necessary by default.** Non-treatment access is scoped to the data needed for the approved purpose.
4. **Part 2 segmentation.** SUD-related records can carry the `SUD_PART2` sensitivity label and be governed independently from ordinary behavioral-health PHI.
5. **Psychotherapy-note segmentation.** `PSYCHOTHERAPY_NOTE` is distinct from normal progress notes.
6. **Signed records are append-only.** Corrections create amendments; they do not silently overwrite signed clinical documentation.
7. **Every clinical fact has provenance.** Source system, author/import actor, event time, version, and integrity hash are modeled.
8. **Audit is a first-class object.** Read, create, update, disclose, export, print, sign, amend, and break-glass actions generate audit events.
9. **No live PHI in source control.** Only synthetic examples may appear under `examples/`.
10. **Compliance is operational, not semantic.** An ontology can support compliance but does not by itself make a system HIPAA- or Part-2-compliant.

## Clinical object model

### Identity and enrollment

- `Patient` — opaque clinical identity keyed by UUID.
- `PatientIdentity` — sealed direct identifiers, stored only in the identity vault.
- `ProgramEnrollment` — admission/enrollment status and program relationship.
- `Referral` — referral source, requested service, screening status, disposition.
- `EpisodeOfCare` — bounded behavioral-health or recovery service episode.

### Care delivery

- `Encounter` — dated service event with start/end, modality, location class, participants, and service code.
- `ProgressNote` — BIRP/SOAP/DAP/other note, with versioning and signature state.
- `ServicePlan` — active plan with goals, objectives, target dates, and review cadence.
- `Goal` / `Objective` — measurable treatment/recovery goals.
- `Intervention` — intervention performed during an encounter.
- `CareCoordinationTask` — appointment, referral, benefits, housing, transportation, or provider-coordination activity.

### Clinical state

- `Diagnosis` — coded diagnosis/problem entry; use licensed/authorized coding systems as appropriate.
- `Assessment` — structured clinical or functional assessment.
- `RiskAssessment` — suicide/self-harm, violence, withdrawal, acute psychiatric, medical, or other safety screening.
- `MentalStatusObservation` — orientation, thought process/content, perception, mood/affect, and related findings.
- `SubstanceUseObservation` — recovery/SUD observation; may be tagged `SUD_PART2`.
- `MedicationStatement` — medication known to be taken or reported.
- `Appointment` — scheduled or completed medical/behavioral-health appointment.

### Workforce and organizations

- `Practitioner` — clinician, QMHP, case manager, CPRS/peer specialist, supervisor, prescriber, or other workforce member.
- `Organization` — Born2BGreat, payer, referring organization, external provider, or partner.
- `Facility` — service location.
- `CareTeam` — workforce relationship to a patient/episode.

### Privacy, governance, and provenance

- `ConsentDirective` — patient consent/authorization scope, recipients, purposes, expiration, and revocation.
- `DisclosureEvent` — outbound disclosure/export and its legal/policy basis.
- `AccessDecision` — authorization result with policy inputs.
- `AuditEvent` — immutable activity event.
- `ProvenanceRecord` — source, author/importer, timestamps, version, and integrity evidence.
- `Signature` — signer, credential, signed timestamp, document version, and document hash.
- `Amendment` — append-only correction to a signed clinical artifact.
- `DeIdentificationRecord` — records Safe Harbor or Expert Determination processing without storing the original identifiers in the public/analytics layer.
- `RetentionRule` — jurisdiction- and record-type-specific retention rule reference.

## Relationship graph

```text
PatientIdentity --IDENTIFIES--> Patient
Patient --HAS_ENROLLMENT--> ProgramEnrollment
Patient --HAS_EPISODE--> EpisodeOfCare
Referral --INITIATES--> EpisodeOfCare
EpisodeOfCare --HAS_SERVICE_PLAN--> ServicePlan
ServicePlan --HAS_GOAL--> Goal
Goal --HAS_OBJECTIVE--> Objective
EpisodeOfCare --HAS_ENCOUNTER--> Encounter
Encounter --DOCUMENTED_BY--> ProgressNote
Encounter --USES_INTERVENTION--> Intervention
Encounter --ASSESSES--> Assessment
Assessment --PRODUCES--> RiskAssessment
Patient --HAS_DIAGNOSIS--> Diagnosis
Patient --HAS_MEDICATION--> MedicationStatement
Patient --HAS_APPOINTMENT--> Appointment
CareTeam --ASSIGNED_TO--> EpisodeOfCare
ConsentDirective --GOVERNS--> ClinicalResource
DisclosureEvent --DISCLOSES--> ClinicalResource
AuditEvent --ACTED_ON--> ClinicalResource
ProvenanceRecord --DESCRIBES--> ClinicalResource
Signature --ATTESTS_TO--> ProgressNote
Amendment --AMENDS--> ProgressNote
```

## Progress-note model

The ontology supports the structure already used in Born2BGreat documentation without requiring any live client data in this repo:

```text
ProgressNote
  noteStyle: BIRP | SOAP | DAP | OTHER
  behaviorSection
  interventionSection
  responseSection
  planSection
  servicePlanGoalRefs[]
  riskAssessmentRef
  mentalStatusObservationRef
  encounterRef
  authorRef
  coSignerRefs[]
  status: DRAFT | SIGNED | AMENDED | ENTERED_IN_ERROR
  version
  signedAt
  provenanceRef
  sensitivity[]
```

## Sensitivity labels

| Label | Meaning |
|---|---|
| `PUBLIC` | Public-source or non-sensitive schema content |
| `INTERNAL` | Internal operational data without PHI |
| `PHI` | Protected health information |
| `E_PHI` | Electronic PHI |
| `DIRECT_IDENTIFIER` | Name, DOB, address, phone, email, government/benefit identifiers, etc. |
| `BEHAVIORAL_HEALTH` | Behavioral-health clinical content |
| `SUD_PART2` | Substance-use-disorder record subject to Part 2 when applicable |
| `PSYCHOTHERAPY_NOTE` | Psychotherapy-note content segmented from ordinary progress notes |
| `HIGHLY_SENSITIVE` | Additional restricted clinical data requiring policy overlay |
| `DEIDENTIFIED` | De-identified data with de-identification provenance |
| `LIMITED_DATASET` | Limited data set governed by appropriate agreement/policy |

## Access decision inputs

Authorization should evaluate all of the following, not role alone:

```text
actor.role
actor.active
actor.organization
careTeamRelationship
purposeOfUse
resource.sensitivity[]
consentOrAuthorization
part2PolicyState
psychotherapyNotePolicyState
minimumNecessaryScope
emergencyBreakGlass
requestContext
```

Suggested workforce roles include `CLINICAL_SUPERVISOR`, `QMHP`, `CASE_MANAGER`, `PEER_RECOVERY_SPECIALIST`, `PRESCRIBER`, `BILLING`, `COMPLIANCE`, `PRIVACY_OFFICER`, `SYSTEM_ADMIN`, `RESEARCH_ANALYST`, and `EXTERNAL_PROVIDER`. Exact permissions must be approved by Born2BGreat privacy/security governance.

## Storage zones

| Zone | May contain live PHI? | Purpose |
|---|---:|---|
| Public GitHub / Green House repo | **No** | Ontology, schema, code, synthetic examples |
| Identity Vault | Yes | Direct identifiers and identity linkage |
| Clinical Graph | Yes | Encounters, notes, diagnoses, plans, assessments |
| Clinical Document Store | Yes | Signed documents and attachments |
| Immutable Audit Store | Minimal PHI only | Access/disclosure/security audit trail |
| De-identified Analytics | No direct PHI | Reporting, operations, research where permitted |

## Standards alignment

The model is intentionally compatible with common healthcare vocabularies and exchange concepts:

- HL7 FHIR concepts such as Patient, Encounter, CarePlan, Goal, Observation, Condition, MedicationStatement, Consent, Provenance, and AuditEvent.
- W3C PROV for provenance concepts.
- ICD-10-CM / SNOMED CT for diagnosis/problem coding where authorized.
- LOINC for coded assessments/observations where appropriate.
- RxNorm for medications.
- CPT/HCPCS for services/billing where applicable.

This repository does not redistribute proprietary code-set definitions.

## Files

- `ontology.jsonld` — core vocabulary and relationships.
- `policy.yaml` — privacy/sensitivity/access-policy vocabulary.
- `shapes.ttl` — SHACL validation rules for key invariants.
- `examples/synthetic-progress-note.jsonld` — fictional example only.

## Production gate

Before any live Born2BGreat instance is ingested, require documented approval that the production environment has appropriate BAAs/contracts where required, access control, authentication, encryption, audit logging, backup/recovery, incident response, risk assessment, workforce policy, consent/Part-2 handling, and jurisdiction-specific retention/disclosure rules.
