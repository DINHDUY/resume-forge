# Prototype

Prototype with all 4 agents, the loop logic, and a polished pipeline UI.

**4 Live AI Agents running in sequence:**
- 📋 **JD Analyst** — extracts required skills, ATS keywords, culture signals, and JD red flags
- 🔍 **Resume Analyst** — scores the resume (0–100) against the JD, finds gaps and keyword misses
- ✍️ **Resume Writer** — rewrites the resume using JD language, adds keywords, quantifies bullets — *without fabricating anything*
- 🚨 **Red Flag Detector** — checks for keyword stuffing, fabrication risks, match issues, then decides: **APPROVE** or **REVISE**

**Loop logic:** If the Red Flag Detector returns REVISE, the pipeline loops back to the Writer with specific fix instructions — up to 3 iterations. If it still fails, it escalates for human review.

**3 tabs to explore:**
- ⚡ **Live Feed** — watch each agent stream its output in real time; click any completed agent to re-read its output
- 📄 **Final Resume** — the optimized resume with a download button
- 🔁 **Loop History** — see the red flag reports from each iteration

It comes pre-loaded with a sample JD and weak resume so you can hit **▶ LAUNCH PIPELINE** immediately.