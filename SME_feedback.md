# SME Feedback: Scaling Strategies for AI Systems
**Module:** Week 5, Class 10 — Scaling Strategies for AI Systems  
**Reviewer:** Sumit Rahman  
**Documents Reviewed:** README.md (student-facing), Instructor Guide

---

## Overall Assessment

This module contains a clear conceptual content in Unit 3 so far. The AI-specific framing of scalability (GPU constraints, model weight loading, inference latency vs. throughput) meaningfully differentiates the lesson from a generic Kubernetes scalingtutorial. The trade-off triangle (Performance / Cost / Reliability) is the strongest
framework introduced in the course and maps directly to the analytical notebook. TheHPA YAML example is accurate and production-realistic.

The primary issues are a significant lab scope mismatch and the recurring placeholder. The stated lab objective — "Containerise a web app using Docker" — describes Lab 3 and Lab 8 work already completed, not the scaling content from this lesson. The lab materials produced alongside this feedback align the lab to the lesson's actual LOs.

---

## 1. Structural Issues

**Issue 1.1 — Lab Objective Misaligned with Lesson Content (High Priority)**

The lesson teaches HPA, VPA, resource limits, cost/performance trade-offs, and `kubectl top`. The stated lab objective is: *"Containerise a web app using Docker to ensure consistent execution across environments."*

This is the objective of Lab 3 (Week 2, Class 3) and partially of Lab 8 (Week 4, Class 8). Students arriving at this lab will have already containerised the NovaMart API twice. The lab materials produced alongside this feedback align with the lesson: the primary deliverables are an optimised multi-stage Dockerfile (a new containerisation
concept not yet covered — build size reduction) plus an HPA configuration and a ResourceQuota — both of which are lesson-specific.


**Issue 1.2 — Section Numbering: Section 3 Labelled "Objective 3"**

The heading for Section 3 in the body reads "Objective 3: Assessing Trade-Offs (Cost vs. Performance)" — using "Objective" rather than a section label. This is an inconsistency with the rest of the document and should be corrected to "3. Assessing Trade-Offs (Cost vs. Performance)".

---

## 2. Alignment with Syllabus and Competency Map

| Learning Objective | In README? | In Instructor Guide? | Notes |
|---|---|---|---|
| Identify key factors influencing AI scalability | ✅ Section 1 | ✅ Yes | GPU/model size framing is AI-specific and strong |
| Configure horizontal and vertical scaling mechanisms | ✅ Section 2 | ✅ Yes | HPA YAML is accurate |
| Assess trade-offs between scalability, performance, and cost | ✅ Section 3 | ✅ Yes | Trade-off triangle is the best framework in Unit 3 |

All three LOs are addressed. The weakest is LO 2 — the VPA is named but not shown
with a YAML example or explained mechanically (how VPA adjusts resource requests
without pod restarts, or with disruption via pod eviction depending on mode).

---

## 3. Technical Content

**Issue 3.1 — VPA Not Explained Mechanically**

Section 2 names the VPA alongside the HPA but gives no YAML example or explanation of how it works. The VPA operates in three modes: `Off` (recommendation only), `Initial` (sets resources on pod creation, no restarts), and `Auto` (evicts and restarts pods when recommendations change). For AI workloads, `Initial` mode is
often preferred because `Auto` mode can evict pods during inference, interrupting live requests.

A one-paragraph explanation of VPA modes with a recommendation to use `Initial` for AI inference pods would complete the HPA/VPA comparison.

**Issue 3.2 — `kubectl top` Requires Metrics Server (Not Mentioned)**

The lesson's practical example uses `kubectl top pods` but does not mention that this command requires the Metrics Server to be installed and running. On a default Minikube cluster it is not enabled. Students who try `kubectl top pods` immediately after reading the lesson will see `error: Metrics API not available`.

The fix is one line: `minikube addons enable metrics-server`. The instructor guide's troubleshooting table correctly documents this (`HPA shows <unknown>/70%`) but it should also appear as a prerequisite note in the student-facing content.

**Issue 3.3 — No Mention of Multi-Stage Docker Builds**

The lesson's containerisation discussion covers resource limits and portability but does not mention multi-stage Docker builds — a key technique for AI workloads where the build environment (compiler, CUDA toolkit) is substantially larger than the runtime environment. This is the concept that differentiates Lab 10's Dockerfile
from the Lab 3 and Lab 8 Dockerfiles. A brief callout in Section 1 (or the lab section) connecting image size to pod startup time and therefore HPA responsiveness would make the Dockerfile task meaningful rather than repetitive.

**Issue 3.4 — Model Weight Loading Not Connected to HPA Strategy**

Reflection Question 3 asks about an 80GB GPU model and the HPA — this is the right question but it is only in the reflection, not in the body. The body discusses "inference latency vs. throughput" and "model size" but does not connect model startup latency (weight loading time) to the HPA's scale-up responsiveness. A pod that takes
2 minutes to load weights is effectively invisible to the HPA for those 2 minutes — it passes the readiness probe only after weights are loaded, meaning the HPA's scale-up event provides no relief for the 2-minute window. This is the key AI-specific HPA complication and deserves a sentence in Section 2.

---

## 4. Instructor Guide

**Issue 4.1 — Live Demo: Generating Load for HPA**

The pacing guide says to "generate load to watch pods scale out dynamically" but does not specify how. The AI Prompt Bank suggests a Python `multiprocessing` script for CPU load, which will work but requires installation of a Python package in the cluster pod — a fragile approach. The lab materials include a `scripts/load_generator.py` that
sends HTTP requests to the service from outside the cluster, which is cleaner and more realistic. The instructor guide should reference this script.

**Issue 4.2 — No Discussion of Scale-Down Lag in Teaching Tips**

The scale-down stabilisation window (5-minute lag visible in the simulation data) is one of the most counter-intuitive HPA behaviours for students who expect elastic infrastructure to be perfectly reactive. The instructor guide should call out that scale-down intentionally lags to prevent flapping — and that this lag has a cost that
engineers must consciously accept or tune.

---

## 5. Theory-to-Lab Alignment Recommendations

**Recommendation 5.1 — Connect Multi-Stage Builds to HPA Speed**  
Frame the multi-stage Dockerfile task as a reliability improvement, not just a disk-space optimisation: smaller images pull faster from the registry, which reduces pod startup time, which means the HPA's scale-up events deliver relief faster. This makes Task 1 (Dockerfile) and Task 2 (HPA) part of the same performance story.

**Recommendation 5.2 — Add Metrics Server Prerequisite to Student README**  
One line: `minikube addons enable metrics-server` should appear in the setup section so students can use `kubectl top` before the HPA demo.

**Recommendation 5.3 — Add VPA `Initial` Mode Note**  
A one-sentence recommendation to use VPA `Initial` mode for AI inference pods (to avoid pod eviction during live inference) would complete the HPA/VPA comparison and directly address the reliability concern the lesson raises around OOMKilled pods.

---

## Summary of Action Items

| Priority | Issue | Type | Owner |
|---|---|---|---|
| High | Fix lab objective — align to scaling LOs, not "containerise a web app" | Fix | Content Author |
| High | Add Metrics Server prerequisite note to student content | Fix | Content Author |
| Medium | Add multi-stage Docker build callout (image size → pod startup → HPA speed) | Recommendation | Content Author |
| Medium | Explain VPA modes (`Off`/`Initial`/`Auto`) with AI-specific guidance | Fix | Content Author |
| Medium | Connect model weight loading time to HPA responsiveness in Section 2 | Recommendation | Content Author |
| Medium | Reference load_generator.py in instructor guide pacing | Fix | Content Author |
| Medium | Add scale-down lag teaching note to instructor delivery tips | Recommendation | Content Author |
| Low | Fix "Objective 3" heading to "3. Assessing Trade-Offs..." | Fix | Content Author |
