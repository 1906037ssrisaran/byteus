# ShieldIQ X

**The Autonomous Financial Crime Investigation Platform**  
From Fraud Alerts to Autonomous Investigations · **Team Byte Us** · Google Cloud AI Builder Cup

ShieldIQ X is a security-conscious, explainable fraud-investigation workspace. Analysts can review risk signals, connect related entities, inspect evidence, request evidence-cited Gemini analysis, simulate scenarios, and prepare compliance drafts behind human approval gates.

> **Implementation status:** Phases 2–13 have code and documentation in this repository. The app is an MVP/demo implementation, not a production-certified banking system. Local authentication uses synthetic demo roles; live Firebase/Vertex AI and Google Cloud deployment require project configuration and credentials. Builds, tests, and cloud deployment have not been certified in this environment.

## Implemented workflows

- **Identity and access:** Firebase ID-token/session-cookie verification; server-managed tenant and role profiles; admin, fraud analyst, compliance officer, and risk manager permissions.
- **Executive command center:** case/alert overview, metrics, trend visualizations, and synthetic-data freshness labeling.
- **Investigation workbench:** case creation and assignment/status updates, evidence, transactions, timeline, analyst notes, and version conflict handling.
- **Relationship intelligence:** interactive entity graph with entity/relationship filtering, risk emphasis, clusters, and evidence provenance.
- **Explainable detection:** versioned transparent rules score only reason signals that map to case evidence; unsupported and unmapped signals are explicitly surfaced. Scores prioritize review—they do not establish fraud.
- **Gemini investigator copilot:** Vertex AI-backed case questions, explanations, briefings, and compliance drafts with structured output and evidence-citation validation.
- **Compliance workflow:** draft, reviewer approval/rejection, immutable version snapshots, and approved-only JSON/HTML/PDF export.
- **Risk simulation:** explicitly heuristic scenario math plus optional Gemini interpretation when Vertex AI is configured. It is not a calibrated predictive model or loss forecast.
- **Operations:** structured request/model logging, health route, rate limits, synthetic seeding, Firestore rules/indexes, BigQuery schemas, GCS, Cloud Run build/deploy configuration, and Terraform infrastructure.

## Repository layout

```text
apps/
  api/                 FastAPI routes, security, domain services, tests, seed utility
  web/                 Next.js 15 analyst experience
docs/                  Product, architecture, data, security, deployment, and demo docs
infra/
  scripts/             PowerShell deployment entry point
  terraform/            GCP resources, IAM, datasets, alerts, and service identities
packages/contracts/    Shared contract workspace
cloudbuild.yaml        Container build and Cloud Run deployment configuration
firestore.rules        Deny-by-default client access rules
firestore.indexes.json Firestore composite indexes
firebase.json          Firebase CLI rules/index configuration
```

## Requirements

- Node.js 20+ and npm.
- Python 3.11+.
- For live auth/model/cloud services: a Firebase project, Google Cloud project, Firebase Auth enabled, Vertex AI access, ADC, and the required IAM grants.
- For infrastructure deployment: Google Cloud CLI, Terraform 1.7+, Firebase CLI, Cloud Build permissions, and a pre-created versioned Terraform state bucket.

## Local development

Install frontend and API dependencies:

```powershell
npm install
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -e "apps\api[dev]"
```

Start both services from the repository root:

```powershell
npm run dev
```

The local API uses an in-memory synthetic tenant and demo-role headers. This mode is permitted only with `SHIELDIQ_ENV=local`; API production configuration rejects demo mode and ignores demo identity headers. The browser demo role selector is a convenience, **not authentication**. Do not use demo mode or synthetic credentials for real data.

- Web: `http://localhost:3000`
- API health: `http://localhost:8000/api/v1/health`
- API OpenAPI: `http://localhost:8000/docs` in local development only
- API tests: `npm run test:api`
- Web checks: `npm run typecheck:web`, `npm run lint:web`, `npm run build:web`

To use real Firebase locally, copy `.env.example` to `.env`, disable both demo flags, supply the Firebase public web configuration and project IDs, create Firebase Auth users, and provision each server-managed `users/{uid}` Firestore profile. The API role and tenant come from that profile—not a browser-selected value. To call Gemini, configure Vertex AI and ADC for the API identity. Never commit `.env` or credentials.

## Synthetic demo data

Local demo mode uses bundled in-memory synthetic fixtures and requires no seeding. To seed a dedicated non-production GCP/Firebase project, use the controlled utility only with ADC and `--confirm-synthetic-only`; create the Firebase Auth test user first:

```powershell
$env:SHIELDIQ_ENV = "staging"
$env:GOOGLE_CLOUD_PROJECT = "your-project-id"
Set-Location apps\api
python scripts\seed_demo.py --confirm-synthetic-only `
  --user "FIREBASE_UID:exact-auth-email:fraud_analyst"
```

The utility blocks production, checks the supplied Firebase UID/email, and writes synthetic records to Firestore and BigQuery. See [Deployment](docs/Deployment.md) and [Demo script](docs/DemoScript.md).

## Google Cloud deployment

Review [Deployment](docs/Deployment.md) first. The deployment script applies Terraform and submits Cloud Build; it creates billable resources and requires explicit project, region, globally unique bucket names, IAM setup, Firebase configuration, and a remote state bucket.

```powershell
.\infra\scripts\deploy.ps1 `
  -ProjectId "your-project-id" `
  -Region "us-central1" `
  -EvidenceBucket "your-globally-unique-evidence-bucket" `
  -TerraformStateBucket "your-precreated-versioned-state-bucket" `
  -FirebaseWebApiKey "your-firebase-web-api-key" `
  -FirebaseWebAppId "your-firebase-web-app-id" `
  -FirebaseAuthDomain "your-project.firebaseapp.com" `
  -WebOrigin "https://your-web-origin"
```

The script does not create or provision Firebase Auth users, assign tenant roles, tune institutional retention policy, or certify a production architecture. Complete the manual setup and security review in the deployment guide before exposing real data.

## Product boundaries

Generated content and scenario scores are decision support. They do not establish guilt, block payments, submit SARs, or replace institutional controls. An authorized human must verify evidence, assess applicable policy, and approve any compliance export.

## Documentation

- [Current AI Builder Cup submission dossier](Submission/Documentation.md)
- [Generated submission package](Submission/)
- [Project overview for UI/UX design assistants](docs/ProjectOverviewForUIDesign.md)
- [Product requirements](docs/PRD.md)
- [System architecture](docs/Architecture.md)
- [Database design](docs/DatabaseSchema.md)
- [API specification](docs/API-Specification.md)
- [AI agent architecture](docs/AgentArchitecture.md)
- [Security architecture](docs/Security.md)
- [Design system](docs/DesignSystem.md)
- [UI audit and redesign notes](docs/UIReview.md)
- [User stories](docs/UserStories.md)
- [Hackathon presentation strategy](docs/PitchDeck.md)
- [Demo script](docs/DemoScript.md)
- [Cloud deployment](docs/Deployment.md)
- [Implementation roadmap](docs/Roadmap.md)
- [Validation checklist, smoke tests, UAT, and bug template](docs/ValidationChecklist.md)
- [Release readiness audit and prioritized remediation](docs/ReleaseReadiness.md)
- [Earlier release readiness assessment snapshot (superseded)](docs/ReleaseReadinessAssessment.md)
