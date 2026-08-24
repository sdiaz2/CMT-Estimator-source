name: CMET Estimator Build
version: 1.0.0
description: >
Full-stack CMET estimating system with OCR, AI classification, cost engine,
anomalies, corrections, forecasting, risk, complexity, reviewer analytics,
plan-set manifest, versioning, and change-impact analysis.

stack:
language: TypeScript
framework: Next.js (App Router)
orm: Prisma
db: PostgreSQL

env:
required:
- DATABASE_URL
optional:
- OCR_ENGINE
defaults:
OCR_ENGINE: tesseract

dependencies:
npm:
- next
- react
- react-dom
- prisma
- @prisma/client
- sharp
- pdf-parse
- tesseract.js
- zod
- uuid

commands:
setup:
- npm init -y
- npm install next react react-dom prisma @prisma/client
- npm install sharp pdf-parse tesseract.js zod uuid
- npx prisma init
migrate:
- npx prisma migrate dev --name init_cmet_estimator
dev:
- npm run dev

project_structure:
root:
- package.json
- next.config.js
- prisma/schema.prisma
- lib/
- app/

files:
prisma/schema.prisma:
action: create
content_hint: |
Define models for:
- User (Reviewer)
- PlanSheet
- PlanSetManifest
- PlanSetVersion
- PlanSheetDelta
- PlanSheetError
- PlanSheetCorrection
- Project
- ProjectCostSummary
- CostAnomaly
- CostCorrection
- ProjectHistory
- ForecastFeedback
- ProjectRisk
- ProjectComplexity
- ReviewerPerformance
- ReviewerIncentive
- PlanSetImpact

lib/prisma.ts:
action: create
content_hint: |
Import { PrismaClient } from '@prisma/client';
Export a singleton prisma client.

lib/ocrEngine.ts:
action: create
content_hint: |
Functions:
- runOCR(buffer): returns extracted text using tesseract.js or pdf-parse.

lib/sheetClassifier.ts:
action: create
content_hint: |
classifySheet(name, text): returns { discipline, confidence }.

lib/planParser.ts:
action: create
content_hint: |
extractQuantities(text): returns { quantities, confidence } for CY, SF, LF, EA.

lib/errorDetector.ts:
action: create
content_hint: |
detectErrors(sheet): returns array of error objects (type, message, severity).

lib/batchOcrProcessor.ts:
action: create
content_hint: |
processBatch(projectId, files):
- uses manifestBuilder + versioning
- runs OCR, classification, quantity extraction
- stores PlanSheet, PlanSheetError
- updates PlanSetManifest stats.

lib/manifestBuilder.ts:
action: create
content_hint: |
buildManifest(projectId, files): creates PlanSetManifest and indexed PlanSheet records.

lib/versioning.ts:
action: create
content_hint: |
createNewVersion(projectId, files): creates PlanSetVersion + manifest linkage.

lib/deltaDetector.ts:
action: create
content_hint: |
detectSheetDelta(oldSheet, newSheet): returns field-level deltas.

lib/versionDeltaProcessor.ts:
action: create
content_hint: |
generateVersionDeltas(projectId, newVersionId): compares sheets and stores PlanSheetDelta.

lib/costAnomalyEngine.ts:
action: create
content_hint: |
detectCostAnomalies(results, rate, labFee): returns array of CostAnomaly-like objects.

lib/costCorrectionEngine.ts:
action: create
content_hint: |
suggestCostCorrections(results, rate, labFee): returns array of CostCorrection-like objects.

lib/projectForecast.ts:
action: create
content_hint: |
forecastProjectCost(params): rule + historical blended forecast with confidence.

lib/forecastFeedback.ts:
action: create
content_hint: |
recordForecastFeedback(projectId, forecastTotal, actualTotal, params).

lib/projectRiskEngine.ts:
action: create
content_hint: |
scoreProjectRisk(projectId): computes ProjectRisk from confidence, errors, anomalies, feedback.

lib/projectComplexityEngine.ts:
action: create
content_hint: |
scoreProjectComplexity(projectId): computes ProjectComplexity from structure, use, sheets, deltas, anomalies, risk.

lib/changeImpactEngine.ts:
action: create
content_hint: |
computeChangeImpact(projectId, oldVersion, newVersion): returns PlanSetImpact.

lib/reviewerPerformanceService.ts:
action: create
content_hint: |
computeReviewerPerformance(projectId): aggregates reviewer metrics.

lib/reviewerIncentiveEngine.ts:
action: create
content_hint: |
computeReviewerIncentive(projectId): computes ReviewerIncentive with breakdown.

api_routes:
• path: /app/api/plans/batch-ocr/route.ts
method: POST
purpose: Run batch OCR, build manifest + version, store sheets, errors.
• path: /app/api/manifest/route.ts
method: GET
purpose: Return PlanSetManifest with sheets for projectId.
• path: /app/api/versions/route.ts
method: GET
purpose: List PlanSetVersion for projectId.
• path: /app/api/versions/deltas/route.ts
method: GET
purpose: Return PlanSheetDelta for projectId + version.
• path: /app/api/project/forecast/route.ts
method: POST
purpose: Return forecastProjectCost(params).
• path: /app/api/project/actual/route.ts
method: POST
purpose: RecordForecastFeedback and update history.
• path: /app/api/project/risk/route.ts
method: GET
purpose: Return scoreProjectRisk(projectId).
• path: /app/api/project/complexity/route.ts
method: GET
purpose: Return scoreProjectComplexity(projectId).
• path: /app/api/project/impact/route.ts
method: GET
purpose: Return computeChangeImpact(projectId, oldVersion, newVersion).
• path: /app/api/reviewers/workload/route.ts
method: GET
purpose: Reviewer workload stats.
• path: /app/api/reviewers/performance/route.ts
method: GET
purpose: Reviewer performance stats.
• path: /app/api/reviewers/incentives/route.ts
method: GET
purpose: Reviewer incentive scores.

pages:
• path: /app/manifest/page.tsx
purpose: Show PlanSetManifest + sheet index.
• path: /app/versions/page.tsx
purpose: Show versions + load deltas.
• path: /app/project/risk/page.tsx
purpose: Show ProjectRisk and drivers.
• path: /app/project/complexity/page.tsx
purpose: Show ProjectComplexity and drivers.
• path: /app/project/impact/page.tsx
purpose: Show PlanSetImpact (cost, trips, quantities, anomalies, risk).
• path: /app/review/workload/page.tsx
purpose: Show reviewer workload dashboard.
• path: /app/review/performance/page.tsx
purpose: Show reviewer performance analytics.
• path: /app/review/incentives/page.tsx
purpose: Show reviewer incentive scoring.

build_flow:
• step: Initialize project
actions: 
◦ run: setup
• step: Define Prisma schema
actions: 
◦ create: prisma/schema.prisma
• step: Generate Prisma client
actions: 
◦ run: npx prisma generate
• step: Implement lib modules
actions: 
◦ create: all files under lib/ with content_hints
• step: Implement API routes
actions: 
◦ create: all files under app/api/ with described behavior
• step: Implement dashboards
actions: 
◦ create: all files under app/*/page.tsx with described behavior
• step: Run migrations
actions: 
◦ run: migrate
• step: Start dev server
actions: 
◦ run: dev

notes_for_agent:
• Ensure all Prisma relations are consistent with hints.
• Use TypeScript for lib and route files.
• Use Next.js App Router conventions (export GET/POST handlers).
• For any “content_hint”, implement minimal but functional logic matching description.
• After build, test flow: 
A. Create project record.
B. Upload plan set via batch-ocr route.
C. Verify manifest, versions, deltas.
D. Run cost evaluation + anomalies + corrections.
E. Call forecast, risk, complexity, impact endpoints.
F. Verify reviewer dashboards.
