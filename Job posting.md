#### [[Dynamic Typing Tool]] and Survey Program Developer ([[Decipher]]/[[Forsta]] + Dashboard)

**Summary**

We are seeking a developer to build (and ideally support ongoing) a [[dynamic “typing tool” algorithm]] and [[sample management system]] embedded in a survey program. The end state is a [[sustainable]], [[templated survey codebase]] that non-technical research analysts can configure for new [[studies]], including [[complex MaxDiff]]. A [[client-ready ROI dashboard]] is required as a deliverable. Current stack: [[Decipher (Forsta)]] for survey programming and [[DisplayR]] for dashboards, but we are open to alternatives if you can make a clear case for [[maintainability]], cost, and speed. Primary [[objectives]]: 1. Survey program codebase that is [[sustainable]] and [[modular]]: We have an existing [[Decipher XML foundation]] and want a clean, maintainable architecture (reusable modules, [[consistent naming conventions]], documented [[configuration approach]]) that can be extended [[study-to-study]] with minimal custom programming. 2. Dynamic typing tool algorithm: Implement a rules-based and/or [[probabilistic]] typing engine that assigns respondents into segments in real time, with support for: • Weighted inputs (survey items, indices, or composite scores) • Confidence / probability or distance-to-segment outputs • Tie-breaking logic and ambiguity handling (e.g., “top 2” assignments) • Transparent logging (how the assignment was reached) for QA and troubleshooting • Export-ready variables for downstream analysis and dashboarding 3. Dynamic sample management program: Implement quota and sample-flow logic within the survey environment, including: • Quotas by segment and/or key demographics • Smart routing / termination / redirect logic based on quotas • Real-time monitoring variables (quota state, completion flags, etc.) • Compatibility with typical panel vendor sample flows (entry links, redirects, statuses) 4. Templatize the analyst-facing front end (configuration layer): We need the survey system to be “templatized” so research analysts can configure studies without editing core code. This includes: • A structured configuration file or admin UI approach (where feasible) • Easy substitution of variables, text, lists, and logic parameters • Ability to field multiple versions / waves with controlled changes • Robust support for complex MaxDiff (design file ingestion, task construction, randomization, versioning, scoring outputs) 5. Client-ready ROI dashboard: Create a dashboard (build API to DisplayR, within Forsta/decipher, or other proposed solution) that can ingest the survey outputs and produce client-facing ROI reporting. Dashboard should be built around wireframes already developed.

**Deliverables**

- Working Decipher/Forsta survey program with modular architecture and documentation
- Implemented dynamic typing algorithm with outputs and QA logging
- Implemented sample management / quota system
- Analyst-facing template/configuration approach (with documentation and examples)
- MaxDiff templating capability (including setup guidance and sample files)
- ROI dashboard: design + build + documented data requirements
- Handoff package: documentation, code comments where appropriate, and a “how to run the next study” guide
- What we will provide
- Existing Decipher XML (current working base)
- Example segment definitions and/or typing logic requirements
- Example MaxDiff design inputs (or we will define the expected format)
- Dashboard requirements and sample mockups/wireframes
- Test links / staging environment access as needed

**You will be asked to answer the following questions when submitting a proposal:**

1. Have you personally implemented advanced survey logic inside Decipher (Forsta), including direct XML edits and JavaScript (not GUI-only programming)?
2. Describe experience implementing a typing tool or segmentation inside a survey
3. Describe experience implementingd MaxDiff using external design files, task construction, and output variables
4. Have you implemented dynamic quotas and routing logic in live field environments?
5. Have you built client-ready dashboards from survey data, and if so, in which tools?