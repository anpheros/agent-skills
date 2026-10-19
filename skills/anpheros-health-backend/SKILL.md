---
name: anpheros-health-backend
description: Build health and medical apps on Anpheros, an HL7 FHIR R4 backend with patient consent (OAuth 2.1 / SMART on FHIR), provenance, an AI context API and a free sandbox with 30 synthetic patients. Use when the user wants to store or read medical records, vitals, lab results, medications, allergies or documents, ask a patient for consent, import HL7/CSV lab reports, or give an AI model a patient's context.
---

# Anpheros health backend

Anpheros stores each person's health record as standard HL7 FHIR R4 resources and exposes the same data through a
simple REST API (`/v1`) and strict FHIR (`/fhir/R4`). Ids are identical in both.

- Base URL: `https://platform.anpheros.com` (the key decides sandbox or production)
- Guides: https://developers.anpheros.com/guides/ — OpenAPI: `/docs` — CapabilityStatement: `/fhir/R4/metadata`
- Documentation MCP server (no account, read-only): `https://platform.anpheros.com/mcp/docs`
- SDKs: `npm install @anpheros/sdk` (TypeScript), `anpheros_sdk` (Dart/Flutter)

## Rules (always)

1. **Sandbox first.** Use an `sk_test_` key: it only ever sees the project's synthetic patients. Never put real
   patient data in the sandbox and never write `sk_live_` keys into code, examples or commits; read keys from the
   environment (`ANPHEROS_KEY`).
2. **Never invent codes.** LOINC, SNOMED CT, ICD-10, ICD-11, ATC and RxNorm codes come from the official code system
   or from the terminology search (`GET /v1/search?q=…`, `GET /v1/search/{id}`). If a code is unknown, leave it out.
   Units are UCUM (`mg/dL`, `mmHg`, `kg`).
3. **Consent is mandatory.** An API key sees only patients its project created. To reach a person's existing record,
   ask them through the consent flow (step 4 below). A `403` means "outside the consent": ask for more, never work
   around it. A `401` after a revocation means the access is over.
4. **Say who wrote it.** Every write carries `author_type` (`patient`, `practitioner`, `device`, `import`, `ai`).
   Data written by a model uses `author_type: "ai"` (HL7 AI Transparency labels are added by the platform).
5. **No diagnosis, no treatment advice.** The app may show records, trends and patterns; it must not tell users what
   they have or which medicine to take.
6. Send an `Idempotency-Key` header on writes that may be retried.

## Workflow

1. **Key.** The developer signs in at https://platform.anpheros.com/dashboard/ and presses *Get a sandbox key*.
2. **Read.**
   ```bash
   curl $BASE/v1/patients -H "Authorization: Bearer $ANPHEROS_KEY"
   curl "$BASE/v1/patients/$PID/observations?category=laboratory&limit=10" -H "Authorization: Bearer $ANPHEROS_KEY"
   curl "$BASE/v1/patients/$PID/timeline?from=2026-06-01" -H "Authorization: Bearer $ANPHEROS_KEY"
   ```
3. **Write.**
   ```bash
   curl -X POST "$BASE/v1/patients/$PID/observations" -H "Authorization: Bearer $ANPHEROS_KEY" \
        -H 'content-type: application/json' -H "Idempotency-Key: $(uuidgen)" -d '{
     "code": "2339-0", "display": "Glucose", "category": "laboratory",
     "value": 112, "unit": "mg/dL", "effective_at": "2026-09-19T08:10:00Z", "author_type": "device"}'
   ```
   With the SDK:
   ```ts
   import { Anpheros, apiKey } from '@anpheros/sdk';
   const anpheros = new Anpheros({ auth: apiKey(process.env.ANPHEROS_KEY!) });
   const patient = await anpheros.patients.create({ given: 'Ana', family: 'Example', birth_date: '1980-05-17', gender: 'female' });
   ```
4. **Consent (SMART on FHIR, OAuth 2.1 + PKCE).** Register an application
   (`POST /admin/projects/{id}/applications`), send the person to `GET /oauth/authorize?…&scope=patient/Observation.rs offline_access&purpose=treatment`,
   exchange the code at `POST /oauth/token`, then call the API with `Authorization: Bearer at_…`.
   Discovery: `/.well-known/smart-configuration`. Guide: https://developers.anpheros.com/guides/healthcare-api-patient-consent
5. **AI context.** `POST /v1/context` returns the parts of a record that fit a prompt, with provenance, instead of
   dumping raw resources into the model.
6. **Lab imports.** HL7 v2 ORU and CSV lab reports: https://developers.anpheros.com/guides/import-lab-results-fhir

Before writing code for a topic not covered here, search the guides (the docs MCP server's `search_docs`, or the
guides index) instead of guessing endpoints.
