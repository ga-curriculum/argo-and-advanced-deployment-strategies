# SME Feedback: Argo and Advanced Deployment Strategies
**Module:** Week 5, Class 9 — Argo and Advanced Deployment Strategies  
**Reviewer:** Sumit Rahman  
**Documents Reviewed:** README.md (student-facing), Instructor Guide

---

## Overall Assessment

This is the most conceptually mature module in Unit 3. The GitOps "pull vs. push"
framing is accurate and well-motivated, the Blue-Green vs. Canary distinction is
the clearest explanation in the course, and the lesson correctly identifies
automated rollback as a reliability mechanism rather than a fallback. The
Reflection Questions are the strongest in the course — Question 2 (subtle data drift
vs. hard crash) is exactly the right prompt for an AI-specific canary discussion.

Two substantive technical gaps need addressing before the lab: the AnalysisTemplate
(the automated rollback mechanism) is mentioned in passing but not explained, and the
Rollout YAML example omits the `canaryService`/`stableService` fields required for
traffic splitting. Both are resolved in the lab materials produced alongside this
feedback.

---

## 1. Structural Issues

**Issue 1.1 — Section Numbering Drift (Recurring)**

The Table of Contents lists Section 1 as "Introduction to GitOps for AI Systems"
but no such heading appears in the body. The body begins with Section 1 labelled
"ArgoCD's Role in Continuous Delivery and GitOps" — which is Section 2 in the ToC.
This is the same drift present in Lessons 5, 7, and 8. The body sections are
numbered 1–3 while the ToC shows 1–6 (including the lab and wrap-up). Recommend
fixing the ToC numbering to match the body section labels.

---

## 2. Alignment with Syllabus and Competency Map

| Learning Objective | In README? | In Instructor Guide? | Notes |
|---|---|---|---|
| Describe ArgoCD's role in CI and GitOps | ✅ Section 1 | ✅ Yes | Pull vs. push distinction is clear |
| Implement staged rollouts (Blue-Green & Canary) | ✅ Section 2 | ✅ Yes | YAML example present but incomplete |
| Evaluate automated rollback and version control | ✅ Section 3 | ✅ Yes | Git revert → source of truth is correct |

All three LOs are addressed. The weakest coverage is LO 2 — the Rollout YAML
example is incomplete (missing the service references and analysis attachment
needed to make it functional). Students who try to adapt the lesson's YAML will
find it does not deploy correctly.

---

## 3. Technical Content

**Issue 3.1 — Rollout YAML Missing Required Fields for Traffic Splitting**

The lesson's Rollout YAML example shows:

```yaml
strategy:
  canary:
    steps:
    - setWeight: 10
    - pause: {duration: 1h}
```

This is syntactically valid but functionally incomplete. To actually split traffic,
Argo Rollouts requires either:
- `canaryService` and `stableService` references (service-based splitting), or
- An ingress/service mesh integration

Without these, `setWeight` has no effect — all pods receive equal traffic regardless
of the weight. The lab's `rollout.yaml` includes `canaryService: novamart-api-canary`
and `stableService: novamart-api-stable` with corresponding `service.yaml`. A note
in Section 2 explaining that traffic splitting requires a second service would prevent
students from deploying the lesson's YAML and wondering why the weight has no effect.

**Issue 3.2 — AnalysisTemplate Not Explained**

Section 3 states that "automated strategies can be configured so that if a Canary
fails its health checks, the system automatically aborts the rollout." However, the
mechanism — the `AnalysisTemplate` object — is never named or shown. Students arrive
at the lab's `analysis-template.yaml` with no lecture context for what it does or why
the `successCondition` field uses PromQL result variable syntax.

Adding a short paragraph to Section 3 explaining that an AnalysisTemplate defines
health-check metrics, success conditions, and failure limits — and that the Rollout
YAML references it by name to wire them together — would make the lab's Task 2
approachable without instructor explanation.

**Issue 3.3 — Rollback is Conflated: Argo Abort vs. Git Revert**

Section 3 describes two different rollback mechanisms without distinguishing them:
1. Argo Rollouts automatic abort: scales canary to zero, routes traffic back to stable
2. Git revert: creates a new commit that changes the image tag back to v1.x

These are not the same operation and happen at different layers. The Argo abort is
immediate and automated — it is a runtime safety mechanism. The Git revert is the
permanent fix that keeps Git as the source of truth. The instructor guide's
troubleshooting table correctly captures this distinction ("students confused
Kubernetes rollback with Git rollback") but the lesson body should also address it.

**Issue 3.4 — Typo: "A rofessional" in Section 1**

*"A rofessional managing an AI inference API..."* — missing "p" in "professional".

---

## 4. Instructor Guide

**Issue 4.1 — 40-Minute Live Demo is Impractical as Described**

The pacing guide allocates 40 minutes to "Staged Rollouts (Live Demo)" with the
instruction to: "Update a model version in Git, watch ArgoCD sync, and show traffic
splitting using Argo Rollouts." This sequence involves:
- A Git commit and push
- ArgoCD webhook/poll cycle (30–60 seconds minimum)
- Argo Rollouts controller detecting the change and starting the rollout
- Waiting through a 1-hour pause (as shown in the lesson's YAML example)

A 1-hour pause is obviously incompatible with a 40-minute demo. The lesson YAML should
use `pause: {duration: 30s}` for demo purposes, with a note that production rollouts
use longer windows. Alternatively, the demo should use `kubectl argo rollouts promote`
to manually advance steps rather than waiting. The instructor guide mentions this
plugin in the troubleshooting table but does not position it as the expected demo tool.

**Issue 4.2 — ArgoCD Installation Not Addressed**

The instructor guide assumes students have ArgoCD installed in their cluster but does
not address the installation step. ArgoCD installation on a local Minikube cluster
requires applying a 12MB manifest and waiting for ~15 pods to start. This takes
5–10 minutes and is a common point of confusion. The lab README provides the
installation command; the instructor guide should note this and recommend starting
Minikube + ArgoCD installation at the beginning of the session while the lecture
proceeds.

---

## 5. Theory-to-Lab Alignment Recommendations

**Recommendation 5.1 — Add AnalysisTemplate to Section 3**  
A one-paragraph explanation of the three-metric AnalysisTemplate (latency, error rate,
confidence) with the `successCondition` syntax would scaffold Task 2 of the lab. The
confidence score metric is AI-specific — worth calling out explicitly as something
traditional SRE pipelines do not include.

**Recommendation 5.2 — Shorten Lesson YAML Pause Duration for Classroom Use**  
Change `pause: {duration: 1h}` to `pause: {duration: 30s}` in the lesson's example
YAML (or add a note explaining the duration is shortened for demonstration). Students
who copy the lesson YAML verbatim will have rollouts that sit paused for an hour.

**Recommendation 5.3 — Connect Canary to Error Budget (Lab 4 Callback)**  
The lesson describes canary as a risk-reduction strategy but does not connect it
to the error budget framework from Lab 4. A sentence noting that an automated rollback
limits error budget consumption to the canary phase window — rather than a full
deployment window — would complete the Unit 2 → Unit 3 continuity.

---

## Summary of Action Items

| Priority | Issue | Type | Owner |
|---|---|---|---|
| High | Add `canaryService`/`stableService` note to Rollout YAML in Section 2 | Fix | Content Author |
| High | Add AnalysisTemplate explanation to Section 3 | Fix | Content Author |
| Medium | Clarify Argo abort vs. Git revert as distinct operations in Section 3 | Fix | Content Author |
| Medium | Fix ToC section numbering drift (recurring issue, 8th module) | Fix | Content Author |
| Medium | Change demo YAML `pause: {duration: 1h}` to `30s` with a note | Fix | Content Author |
| Medium | Address ArgoCD installation time in instructor guide pacing | Fix | Content Author |
| Low | Fix typo: "A rofessional" → "A professional" in Section 1 | Fix | Content Author |
| Low | Add error budget connection to canary risk framing | Recommendation | Content Author |
