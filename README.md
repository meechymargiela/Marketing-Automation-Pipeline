## Marketing Automation Pipeline — HubSpot + Make.com

> **Portfolio Project ** Marketing Automation | Marketing Operations  
> Built by Mitchell Louw | June 2026

---

## Project Overview

A fully production-grade, behaviour-triggered B2B lead nurturing pipeline built in HubSpot Enterprise. The pipeline automates the entire journey from form submission to Marketing Qualified Lead (MQL), with intelligent branching logic, a custom lead scoring model, and a parallel Make.com scenario for Google Sheets logging, and Slack notifications.

**The pipeline runs 44 workflow steps across 7 emails, 6 decision branches, and 3 disqualification exits, entirely automated.**

---

## Tech Stack

| Tool | Role |
|---|---|
| **HubSpot Enterprise** | Workflow automation, email sending, lead scoring, CRM |
| **Make.com** | Form submission listener, Slack notifications, Google Sheets logging |
| **Google Sheets** | Contact data logging and pipeline tracking |
| **Slack** | Real-time internal sales notifications |

---

## Architecture

### Pipeline Flow

```
Form Submission
      │
      ▼
┌─────────────────────┐
│  Disqualification   │ ──── Junk email / competitor domain ──► EXIT (Unqualified)
│  Gate               │
└─────────────────────┘
      │ Clean lead
      ▼
┌─────────────────────┐
│  Email 1            │ Day 0 — Welcome + Quick Win
│  Welcome            │
└─────────────────────┘
      │
      ▼ Wait 3 days
┌─────────────────────┐
│  Opened?            │ ──── No ──► Resend alt subject (Day 4) ──► Still no? ► EXIT
└─────────────────────┘
      │ Yes
      ▼
┌─────────────────────┐
│  Email 2            │ Day 3–4 — Pain Point Education
│  Education          │
└─────────────────────┘
      │
      ▼ Wait 4 days
┌─────────────────────┐
│  Email 3            │ Day 8–11 — Social Proof / Case Study
│  Social Proof       │
└─────────────────────┘
      │
      ▼ Wait 3 days
┌─────────────────────┐
│  Score Gate         │ ──── Score < 25 ──► Slow track (7-day cadence)
│  25pt threshold     │
└─────────────────────┘
      │ Score ≥ 25
      ▼ Internal notification + fast track
┌─────────────────────┐
│  Email 4            │ Day 14–18 — Objection Handler
│  Objection          │
└─────────────────────┘
      │
      ▼ Wait 5 days
┌─────────────────────┐
│  Email 5            │ Day 21–25 — Soft CTA
│  Soft CTA           │
└─────────────────────┘
      │
      ▼ Wait 3 days
┌─────────────────────┐
│  Clicked CTA?       │ ──── Yes ──► MQL CONVERSION ✅
└─────────────────────┘
      │ No
      ▼
┌─────────────────────┐
│  Email 6            │ Day 33–35 — Second Soft CTA
└─────────────────────┘
      │
      ▼ Wait 10 days
┌─────────────────────┐
│  Email 7            │ Day 44–46 — Break-up Email
│  Break-up           │
└─────────────────────┘
      │
      ▼ Wait 5 days
┌─────────────────────┐
│  Re-engaged?        │ ──── Yes ──► Internal notification, manual follow-up
└─────────────────────┘
      │ No
      ▼
EXIT (Unqualified / Cold)
```

---

## Lead Scoring Model

Custom engagement scoring built in HubSpot's Lead Scoring tool. **MQL threshold: 50 points.**

| Behaviour | Points |
|---|---|
| Marketing email opened | +5 |
| Marketing email link clicked | +10 |
| Reply to email | +20 |
| Form submission (additional) | +15 |
| Meeting booked | +25 |

### MQL Conversion Paths

A lead can reach MQL status through multiple engagement combinations, for example:

- Books a meeting (+25) + replies to email (+20) + clicks a link (+10) = **55 pts ✅**
- Fills second form (+15) + clicks 3 links (+30) + opens 2 emails (+10) = **55 pts ✅**

---

## Email Sequence

| # | Name | Day | Goal |
|---|---|---|---|
| 1 | Welcome + Quick Win | 0 | Deliver value immediately, build trust |
| 1B | Alt Subject Resend | 4 | Re-attempt for non-openers |
| 2 | Pain Point Education | 3–4 | Show understanding of their problem |
| 3 | Social Proof / Case Study | 8–11 | Build credibility, reduce risk |
| 4A/4B/4C | Objection Handler | 14–18 | Pre-empt "not sure yet" mindset |
| 5 | Soft CTA | 21–25 | Low-friction conversion ask |
| 6 | Second Soft CTA | 33–35 | Different angle, same offer |
| 7 | Break-up Email | 44–46 | Surface silent intent, create urgency |

**Send window:** Monday–Friday, 8am–5pm (business days only)

---

## Branch Logic

### Disqualification exits
- **Entry gate:** Junk email patterns (info@, support@, admin@, noreply@) or competitor domains → immediately set Unqualified + exit
- **Non-opener exit:** Contact ignores Email 1 AND Email 1B resend → set Unqualified + exit
- **Cold exit:** No response to Email 7 break-up → set Lifecycle = Unqualified, Lead Status = Unqualified, add to cold re-engagement list

### Acceleration branches
- **Email open check:** Opened Email 1 within 3 days → fast track cadence; did not open → resend with alt subject
- **Mid-funnel score gate:** Score ≥ 25 at Email 3 → internal sales notification + 2-day fast track; Score < 25 → 7-day slow cadence
- **CTA click check:** Clicked Email 5 CTA → instant MQL conversion; did not click → continue to Email 6 + 7

### MQL conversion trigger
When Lifecycle Stage is set to **Marketing Qualified Lead:**
1. Lead Status → In Progress
2. Internal email notification fires to sales rep
3. Task created: "Follow up within 24 hours"
4. Contact unenrolled from nurture sequence automatically

---

## ⚙️ Make.com Parallel Scenario

A parallel Make.com scenario handles external data routing, logging lead data to Google Sheets for stakeholder reporting and triggering Slack notifications for real-time sales awareness. Keeping HubSpot clean and focused solely on nurture and scoring logic

```
HubSpot Form Submission
        │
        ▼
  Watch HubSpot Form  
        │
        ├──► Log contact data to Google Sheets
        │         (Name, Email, Company, Date, Source)
        │
        └──► Send Slack notification to team
                  ("New lead: [Name] from [Company]")
```

> **Note:** Email sending is handled entirely by HubSpot for tracking consistency. The Make.com scenario handles Google sheets logging and Slack team notifications only.

---

## 🔧 Setup & Configuration

### HubSpot Requirements
- Marketing Hub Professional or Enterprise (lead scoring requires Pro+)
- Custom sending domain with SPF, DKIM, and DMARC records configured
- Marketing email subscription type configured
- GDPR legal basis set for all contacts (Legitimate Interest or Consent)

### Make.com Requirements
- HubSpot connection authenticated
- Google Sheets connection authenticated
- Slack connection authenticated
- Scenario set to watch specific HubSpot form ID

### Workflow Settings
- Contact-based workflow
- Enrollment trigger: Form submission
- Re-enrollment: Off
- Send window: Mon–Fri, 8am–5pm
- Goal unenrollment: Lifecycle Stage = Marketing Qualified Lead

---

## Repository Structure

```
├── README.md                          # This file
├── docs/
│   ├── workflow-flow-diagram.png      # Full pipeline visual
│   ├── lead-scoring-setup.png         # HubSpot scoring configuration
│   ├── email-sequence-overview.png    # All 7 emails mapped
│   └── make-scenario-diagram.png      # Make.com flow
├── emails/
│   ├── email-1-welcome.html           # Email templates
│   ├── email-2-education.html
│   ├── email-3-social-proof.html
│   ├── email-4-objection.html
│   ├── email-5-soft-cta.html
│   ├── email-6-second-cta.html
│   └── email-7-breakup.html
└── make-scenario/
    └── scenario-blueprint.json        # Exportable Make.com scenario
```

---

## Key Design Decisions

**Why a 25pt mid-funnel score gate**  
Rather than sending the same 7 emails to every lead regardless of engagement, the score gate at Email 3 identifies warm leads early and accelerates them toward the CTA — reducing the time from lead to MQL for high-intent contacts.

**Why a break-up email at step 7**  
Leads who have silently read emails without clicking represent latent intent. A break-up email ("Should I stop sending?") consistently surfaces replies from these contacts at a higher rate than a standard nurture email, creating a re-engagement signal that routes them to manual sales follow-up.

**Why dual-track cadence vs single cadence**  
A single send cadence treats a lead who opened 4 emails identically to one who opened none. The dual-track approach respects engagement signals — fast openers get accelerated, slow contacts get throttled to reduce unsubscribe risk.

---

## Results & Validation

- Pipeline successfully deployed and tested end-to-end in HubSpot Enterprise
- Contact enrollment, lifecycle stage updates, email sends, branch routing, and MQL conversion all validated via HubSpot action logs
- Internal notifications and task creation confirmed firing on MQL trigger
- Make.com scenario validated: Google Sheets logging and Slack notifications

> **Note on deliverability:** Testing was conducted on a HubSpot free sending domain (`hubspotfree.eu1.hs-send.com`). In a production environment, a custom authenticated sending domain with SPF/DKIM/DMARC records would be configured to ensure inbox placement and sender reputation.

---

**Mitchell Louw**  
Digital Marketing & AI Marketing Automation Specialist 
📍 Pretoria, South Africa  
🔗 [LinkedIn](https://linkedin.com/in/your-profile)  
🌐 [Portfolio](https://your-portfolio-site.com)

---

## Screenshots

## Workflow Canvas
<img width="386" height="864" alt="Screenshot 2026-06-10 090021" src="https://github.com/user-attachments/assets/0bf6ecc1-5f8d-4060-ab8c-e57e92e1a128" />
<img width="1909" height="875" alt="image" src="https://github.com/user-attachments/assets/5ef521ab-c120-4ce0-91d8-41aa819af9b1" />
<img width="1898" height="869" alt="image" src="https://github.com/user-attachments/assets/cc298a4f-686a-4e52-8e17-aa6186e7f528" />
<img width="1910" height="882" alt="image" src="https://github.com/user-attachments/assets/bbf19aa7-c6d1-4e76-9772-e3c4c9c8eb31" />
<img width="1906" height="821" alt="image" src="https://github.com/user-attachments/assets/ecbb1232-3e6a-477b-92e2-6aa010d37400" />
<img width="1907" height="876" alt="image" src="https://github.com/user-attachments/assets/942bfa06-5411-49f5-9f5d-197f6817fa2a" />
<img width="1909" height="874" alt="image" src="https://github.com/user-attachments/assets/639a3fc0-cc23-41ee-b244-f09e0478b62f" />
<img width="1903" height="848" alt="image" src="https://github.com/user-attachments/assets/aee15820-66c3-4612-8fef-e717ba899d97" />
<img width="1910" height="878" alt="image" src="https://github.com/user-attachments/assets/f20aa359-e81f-4470-8186-d06d206b4b84" />

## Lead Scoring Setup
<img width="1910" height="826" alt="image" src="https://github.com/user-attachments/assets/ee4be41c-3b04-4bfb-af9b-ce7c34f4f6f4" />
<img width="1909" height="873" alt="image" src="https://github.com/user-attachments/assets/202ba48a-cf78-496c-b937-b55998872553" />

## Enrollment History Successfull Run
<img width="1909" height="928" alt="image" src="https://github.com/user-attachments/assets/869dc2de-3050-472c-a6fa-8f7d364dd833" />
<img width="1908" height="947" alt="image" src="https://github.com/user-attachments/assets/64c676c1-212b-4624-9687-a1671f3c618b" />

## Email Delivered To Inbox
<img width="1903" height="935" alt="image" src="https://github.com/user-attachments/assets/65ce4e6f-d0b2-42c3-96ba-575787c98224" />

## Make.com Scenario

<img width="1910" height="940" alt="image" src="https://github.com/user-attachments/assets/6bc88d1b-c5cd-4eb5-b393-6151067b423e" />

## License

This project is for portfolio demonstration purposes. Email templates and workflow logic may be adapted for commercial use with attribution.
