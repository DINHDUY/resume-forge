# Collaborative AI Agent Pipeline: Resume ↔ Job Description Optimizer

Here's a comprehensive plan for a multi-agent system that evaluates, rewrites, and validates resumes against job descriptions (JD) in a feedback loop.

---

## 🏗️ System Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    ORCHESTRATOR AGENT                       │
│              (Controls flow & loop logic)                   │
└──────┬──────────────────────────────────────────────────────┘
       │
       ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   JD ANALYST │───▶│RESUME ANALYST│───▶│RESUME WRITER │───▶│  RED FLAG    │
│    AGENT     │    │    AGENT     │    │    AGENT     │    │ DETECTOR     │
└──────────────┘    └──────────────┘    └──────────────┘    └──────┬───────┘
       ▲                                                           │
       └─────────────────── LOOP BACK ─────────────────────────────┘
```

---

## 🤖 Agent Definitions

### 1. **Orchestrator Agent**
- **Role:** Master controller — routes inputs, tracks loop iterations, enforces exit conditions
- **Inputs:** Raw JD + Raw Resume
- **Outputs:** Final approved resume + audit report
- **Logic:**
  - Max loop iterations: `3` (configurable)
  - Exit condition: Red Flag score < threshold AND Match score > threshold
  - Tracks version history of each rewrite

---

### 2. **JD Analyst Agent**
- **Role:** Deep-parse the job description
- **Outputs a structured schema:**
```json
{
  "role_title": "...",
  "required_skills": [],
  "preferred_skills": [],
  "experience_years": "",
  "education_requirements": "",
  "soft_skills": [],
  "keywords_for_ats": [],
  "company_culture_signals": [],
  "compensation_signals": "",
  "red_flag_jd_indicators": []  // e.g., "rockstar", "wear many hats"
}
```
- **Prompt focus:** Distinguish between hard requirements vs. nice-to-haves; extract ATS keywords

---

### 3. **Resume Analyst Agent**
- **Role:** Deep-parse the resume and score it against the JD analysis
- **Outputs:**
```json
{
  "candidate_skills": [],
  "experience_summary": [],
  "gaps_vs_jd": [],
  "strengths_vs_jd": [],
  "match_score": 0-100,
  "ats_keyword_coverage": "X/Y keywords found",
  "formatting_issues": [],
  "quantification_gaps": []  // bullets missing numbers/impact
}
```
- **Scoring rubric:** Skills match (40%), Experience relevance (30%), ATS keywords (20%), Formatting (10%)

---

### 4. **Resume Writer Agent**
- **Role:** Rewrite the resume to maximize fit — *without fabricating experience*
- **Rules (system prompt constraints):**
  - ✅ Reframe existing experience using JD language
  - ✅ Prioritize relevant roles/bullets, deprioritize irrelevant ones
  - ✅ Add missing ATS keywords where truthfully applicable
  - ✅ Quantify vague bullets ("Improved performance" → "Improved load time by 40%")
  - ❌ Never invent roles, companies, or skills
  - ❌ Never change dates
- **Outputs:** Rewritten resume (markdown or structured JSON) + change log

---

### 5. **Red Flag Detector Agent**
- **Role:** Dual-sided red flag detection — flags issues in both the **JD** and the **rewritten resume**

| Target | Red Flags Detected |
|--------|-------------------|
| **JD** | Unrealistic requirements, vague comp, toxic culture signals, discriminatory language |
| **Resume** | Fabrication risk, unexplained gaps, keyword stuffing, inconsistent dates, ATS-breaking formatting |
| **Match** | Over-qualification, under-qualification, role mismatch |

- **Outputs:**
```json
{
  "jd_red_flags": [],
  "resume_red_flags": [],
  "match_red_flags": [],
  "severity": "low | medium | high",
  "recommendation": "approve | revise | warn_user"
}
```
- **Loop trigger:** If `severity = high` OR `match_score < 70` → send back to Writer Agent with specific fix instructions

---

## 🔁 The Loop Logic

```
ITERATION 1:
  JD Analyst → Resume Analyst → Resume Writer → Red Flag Detector
                                                      ↓
                                              score < threshold?
                                                YES → feedback to Writer
                                                NO  → EXIT ✅

ITERATION 2 (if looped):
  Writer receives: previous_resume + red_flag_report + jd_analysis
  Targeted rewrite (only fix flagged issues)
                                                      ↓
                                              score < threshold?
                                                YES → ITERATION 3
                                                NO  → EXIT ✅

ITERATION 3 (final):
  Final attempt — if still failing, escalate to human review
```

---

## 🛠️ Technical Implementation Plan

### Stack Recommendation
| Component | Technology |
|-----------|-----------|
| Agent Framework | LangGraph / CrewAI / AutoGen |
| LLM Backend | Claude (Sonnet for writers, Haiku for parsers) |
| State Management | LangGraph StateGraph or Redis |
| Resume Parsing | PyMuPDF / python-docx + LLM |
| Output Format | Structured JSON → rendered PDF/DOCX |
| API Layer | FastAPI |
| Frontend | React dashboard |

### State Object (passed between agents)
```python
class PipelineState:
    job_description: str
    original_resume: str
    jd_analysis: dict
    resume_analysis: dict
    current_resume: str        # mutates each iteration
    rewrite_changelog: list
    red_flag_reports: list     # history across iterations
    iteration: int
    final_match_score: float
    status: str                # "running" | "approved" | "escalated"
```

---

## 📋 Phased Rollout

**Phase 1 — Core Pipeline (Weeks 1–3)**
- Build JD Analyst + Resume Analyst agents
- Establish structured JSON schemas
- Basic orchestrator with linear flow (no loop yet)

**Phase 2 — Writer + Loop (Weeks 4–6)**
- Resume Writer agent with hard guardrails
- Red Flag Detector with scoring thresholds
- Implement loop logic with iteration cap

**Phase 3 — Intelligence Layer (Weeks 7–9)**
- ATS simulation scoring
- Company culture fit analysis
- Salary range red flag detection

**Phase 4 — UX & Productization (Weeks 10–12)**
- User dashboard (upload JD + resume, view rewrite diffs)
- Human-in-the-loop review UI for escalated cases
- Export to PDF/DOCX

---

## ⚠️ Key Guardrails & Ethics

- **Transparency:** Always show users what changed and why
- **Anti-hallucination:** Writer agent must cite which original experience each rewrite derives from
- **User consent:** Warn users when JD has red flags before they apply
- **Bias detection:** Flag JDs with potentially discriminatory language

---

Want me to build a working prototype of this as an interactive artifact, scaffold the code, or dive deeper into any specific agent?