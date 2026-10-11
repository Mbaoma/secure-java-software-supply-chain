# Technical PRD: Secure Java Software Supply Chain Workshop

**Author:** Mbaoma and Success
**Status:** Draft
**Audience:** Java Developer, Cloud Native Engineer

## Why we're building this

Our workshop customers — teams at banks, fintechs, and similarly regulated shops — keep
asking variations of the same question: "how do we know this software is what we think it
is?" Most small teams can't answer that today. They have no record of where a build came
from, no visibility into what's actually inside their containers, and nothing stopping an
unverified image from reaching production.

I want our team to ship a workshop that teaches people to close that gap, using a real
(if small) Java application and a real GCP pipeline. Students should walk away having
personally watched an outdated dependency get flagged and an unsigned image get rejected
— not just read about SLSA and Binary Authorization in the abstract.

## Goal

Deliver a working, teachable pipeline where a Java app is built, scanned, signed, and
deployed, and where Cloud Run enforces that only pipeline-built, pipeline-signed images
can run. The whole thing needs to be simple enough to be a *first* workshop in container
and supply chain security — beginner to intermediate difficulty, not an expert deep dive.

## Success criteria

- A Jakarta EE app (loan-calculator or wallet service) runs on Cloud Run when deployed
  "the normal way," with no scanning or signing involved.
- With vulnerability scanning turned on, the app's deliberately outdated dependency is
  flagged by Artifact Analysis, visibly and understandably.
- With Binary Authorization turned on, an attempt to deploy an unsigned image is rejected
  by Cloud Run in front of the class.
- A student with basic Java and cloud familiarity can get through all three stages in a
  single workshop session without getting stuck on unrelated complexity.

## Scope

**In scope:**
- One small Jakarta EE application, intentionally simple (loan calculator or wallet
  service), running on Eclipse Temurin.
- Cloud Build pipeline that builds the app and emits SLSA provenance.
- Artifact Registry storage + Artifact Analysis vulnerability scanning.
- Binary Authorization policy enforced on Cloud Run.
- A three-stage demo/workshop script: build & run, scan, lock down.

**Out of scope:**
- Multiple services, databases, or production-grade app features — the app is a teaching
  vehicle, not a product.
- Alternative runtimes or non-GCP deployment targets.
- Advanced SLSA levels, key management infrastructure, or signing ceremonies beyond what's
  needed to demonstrate Binary Authorization working.
- Any later/related workshops that build on this one — this PRD covers only the
  beginner/intermediate workshop described here.

## Work breakdown by role

### Java Developer

- Build the Jakarta EE application (loan-calculator or wallet service) — small enough to
  explain in a few minutes, but with enough real logic that it feels like an actual app
  rather than a toy.
- Select and pin the app's dependencies, keeping the dependency tree realistic (i.e.
  representative of what a real Jakarta EE service would pull in).
- Deliberately introduce one outdated library with a known CVE, chosen so Artifact
  Analysis will flag it clearly and the finding is easy to explain to students.
- Package the app for Eclipse Temurin and confirm it runs correctly as a standalone
  container before any pipeline concerns are layered on.
- Make sure the app has no hidden assumptions that would break when rebuilt headlessly by
  Cloud Build (e.g., no reliance on local developer machine state).
- Provide whatever minimal app-level hooks the pipeline needs (health check endpoint,
  predictable container entrypoint, versioned artifact naming) so the cloud-native side has
  a stable integration point.

### Cloud Native Engineer

- Set up Artifact Registry to store built images.
- Write the Cloud Build configuration that builds the app's container image and produces
  SLSA provenance for each build.
- Configure Artifact Analysis vulnerability scanning on images pushed to Artifact
  Registry, and verify that the Java developer's planted outdated library actually
  surfaces as a finding.
- Design and configure the signing step and the Binary Authorization policy so that Cloud
  Run only accepts images signed by the Cloud Build pipeline.
- Deploy the app to Cloud Run and validate end-to-end behavior at each of the three
  workshop stages.
- Script and rehearse the three-stage demo sequence (including deliberately attempting an
  unsigned deploy in stage three) so it's reliable and repeatable live, in front of a
  classroom.

## Milestones and sequencing

These map directly to the three workshop stages — each should be independently
demoable before moving to the next, since that's also the order students will experience
it in:

1. **Build and run.** Jakarta EE app complete, containerized on Eclipse Temurin, deployed
   to Cloud Run with no scanning or signing. Owner: Java Developer builds the app; Cloud
   Native Engineer stands up the baseline Cloud Run deployment.
2. **Scan it.** Cloud Build + Artifact Registry + Artifact Analysis wired together; outdated
   library confirmed to trigger a visible finding. Owner: Cloud Native Engineer wires the
   pipeline; Java Developer confirms the planted dependency behaves as intended.
3. **Lock it down.** Signing and Binary Authorization enforced on Cloud Run; unsigned
   image deploy attempt confirmed to fail. Owner: Cloud Native Engineer.

Each milestone should end with a rehearsed, reliable demo — not just a passing pipeline —
since the whole point is for students to watch it happen live.

## Risks and open questions

- **Demo flakiness:** live rejection of an unsigned image and live flagging of a
  vulnerability both need to be reliable under workshop time pressure. We should rehearse
  these, not assume they'll "just work" on the day.
- **CVE churn:** the deliberately outdated library needs a CVE that Artifact Analysis will
  reliably catch; if the vulnerability database stops flagging it (e.g., it's fixed/delisted
  upstream), we need a process to swap in a new one.
- **App realism vs. simplicity:** the Jakarta EE app needs to be realistic enough that the
  vulnerability scanning stage feels meaningful, but simple enough not to distract from the
  pipeline concepts — this balance needs sign-off from both roles, not just the Java
  developer.
- **Binary Authorization policy scope:** open question on how strict the policy should be
  for teaching purposes (e.g., single attestor vs. multiple) — needs to be strict enough to
  demonstrate real enforcement, simple enough to explain in one workshop.
- **Environment reset between workshop runs:** we haven't yet defined how Artifact
  Registry, Binary Authorization policy, and Cloud Run services get reset between workshop
  cohorts — needs a decision before this scales beyond a single run.
