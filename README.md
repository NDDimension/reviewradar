# 📡 ReviewRadar — AI-Powered Google Review Intelligence Platform

> **Automatically monitor, analyze, and act on Google reviews for any business — powered by AI, delivered as a live dashboard.**

A production-grade reputation intelligence system built on **n8n + Gemini AI + Google Sheets + Looker Studio**. Monitors multiple businesses simultaneously, scores every review with AI, fires multi-channel alerts for critical feedback, and delivers weekly executive reports — all running autonomously, 24/7.

---

![n8n](https://img.shields.io/badge/n8n-Automation-orange?style=for-the-badge&logo=n8n)
![Gemini](https://img.shields.io/badge/Google_Gemini-AI_Analysis-4285F4?style=for-the-badge&logo=google)
![Looker Studio](https://img.shields.io/badge/Looker_Studio-Dashboard-669DF6?style=for-the-badge&logo=looker)
![Google Sheets](https://img.shields.io/badge/Google_Sheets-Data_Warehouse-34A853?style=for-the-badge&logo=googlesheets)
![Slack](https://img.shields.io/badge/Slack-Alerts-4A154B?style=for-the-badge&logo=slack)
![WhatsApp](https://img.shields.io/badge/WhatsApp-Alerts-25D366?style=for-the-badge&logo=whatsapp)
![Twilio](https://img.shields.io/badge/Twilio-WhatsApp_API-F22F46?style=for-the-badge&logo=twilio)

---

## 🎬 Live Demo

[![Watch Demo](https://img.shields.io/badge/Watch_Demo-Loom_Video-625DF5?style=for-the-badge)](https://drive.google.com/file/d/1Xw2v594S6xN3jL53L9E5sBwfLyU7kcGP/view?usp=sharing)

[![Live Dashboard](https://img.shields.io/badge/Live_Dashboard-Looker_Studio-669DF6?style=for-the-badge)](https://datastudio.google.com/reporting/9069f597-1a39-42f7-9435-d58a17297c04)

---

## 🧩 The Problem It Solves

Local businesses — restaurants, clinics, hotels, salons — receive Google reviews constantly. Most owners:

- Discover 1-star reviews **days later**, by accident
- Have **no visibility into patterns**: is "slow service" a recurring complaint every weekend?
- Pay **$100–$500/month** for SaaS tools like ReviewTrackers or Yext to do this
- Have **zero actionable intelligence** — just a raw list of reviews

**ReviewRadar replaces all of that** with a custom-built, AI-powered system the business owns outright — for a one-time setup cost.

---

## ⚙️ System Architecture

```
[Google Maps]
      ↓
[SERPAPI — Review Fetcher]
      ↓
[n8n — Automation Engine]
      ↓
[Gemini AI — Review Analysis]
   ↙      ↓      ↘
[Gmail] [Slack] [WhatsApp]     ← Instant alerts for urgent reviews
      ↓
[Google Sheets — Data Warehouse]
   ↙         ↘
[All Reviews] [Testimonials] [Weekly Summary]
      ↓
[Looker Studio — Live Dashboard]
      ↓
[Weekly AI Report — Email per business]
```

---

## 🔄 Workflows

### Workflow 1 — Review Watcher *(Runs on schedule, fully autonomous)*

1. Reads all monitored businesses from the **Businesses** sheet (Name + SERP Data ID + Owner Email)
2. Fetches latest Google Maps reviews for each business via **SERPAPI**
3. Deduplicates against existing records using **Review ID matching** — skips already-processed reviews
4. For each new review, **Gemini AI** produces structured analysis:
   - `sentiment`: positive / negative
   - `category`: Food / Service / Staff / Cleanliness / Other
   - `urgency`: 1–5 numeric score
   - `draft_response`: AI-written professional owner reply
5. Routes based on urgency:
   - 🔴 **Urgency 4–5 / Rating 1–2:** Fires **3 simultaneous alerts** — Gmail + Slack + WhatsApp (Twilio)
   - 🟢 **Positive / Rating 4–5:** Appends to **Testimonials** sheet
6. All reviews logged to **All Reviews** sheet with full structured data

---

### Workflow 2 — Weekly Intelligence Report *(Every Monday, 8:00 AM)*

1. Reads all reviews from the **All Reviews** sheet
2. Groups and aggregates reviews by business using a JavaScript Code node
3. **Gemini AI** analyzes each business group and produces:
   - Average rating
   - Top 3 recurring complaint themes
   - Top 3 praise themes
   - One specific, actionable recommendation
4. **Business-wise separator** splits output — each business owner gets their own individual report email
5. Results saved to **Weekly Summary** sheet for trend tracking over time

---

### Dashboard — 3-Page Looker Studio *(Live, auto-updating)*

**Page 1 — Executive Dashboard**
- KPI cards: Total Reviews, Average Rating, Businesses Monitored, Urgent Reviews
- Reviews by Business (bar chart)
- Average Rating by Business (comparative bar)
- Review Volume Over Time (time-series)
- Business filter to isolate individual locations

**Page 2 — Customer Insights**
- Sentiment Distribution (pie: positive vs negative)
- Top Review Categories (bar: Food, Service, Staff, Cleanliness, Other)
- Recent Reviews table with full data

**Page 3 — AI Executive Reports**
- Per-business: Top Complaints, Top Praise, AI-generated Recommendations
- Designed for owners and decision-makers

---

## 🛠️ Tech Stack

| Tool | Role |
|------|------|
| n8n | Automation engine (2 workflows) |
| SERPAPI | Google Maps review fetching |
| Google Gemini AI | Review analysis, categorization, report generation |
| Google Sheets | Data warehouse (4 tabs) |
| Gmail | Owner alerts + weekly report delivery |
| Slack | Urgent review channel notifications |
| Twilio | WhatsApp critical review alerts |
| Looker Studio | Live business intelligence dashboard |

---

## 📊 Live Results

Built and tested on real businesses:

| Business | Reviews | Avg Rating | Urgent Alerts |
|----------|---------|------------|---------------|
| Cozy Restaurant | 8 | 3.88 | ✅ Triggered |
| McDonald's (Mehsana) | 8 | 3.88 | ✅ Triggered |

> Sentiment split: 75% positive, 25% negative across 16 reviews

---

## 📁 Repository Structure

```
reviewradar/
│
├── workflows/
│   ├── ReviewRadar_Watcher.json          ← Main review fetching + alerting workflow
│   └── ReviewRadar_Weekly_Report.json    ← Monday report generation workflow
│
├── assets/
│   ├── looker-studio.pdf                 ← PDF of Looker Studio Dashboard
│   └── workflow-canvas.png               ← n8n workflow screenshot
│
├── README.md
└── setup-guide.md
```

---

## 🚀 Quick Setup

See **[setup-guide.md](./setup-guide.md)** for full configuration instructions.

**You will need:**
- n8n account (cloud or self-hosted)
- SERPAPI account + API key (100 free searches/month)
- Google Gemini API key (free at aistudio.google.com)
- Google account (Sheets + Gmail + Calendar)
- Slack workspace (for alert channel)
- Twilio account (for WhatsApp alerts — free trial available)
- Looker Studio account (free with Google account)

---

## 💡 Customization Options

- Add more businesses — just add rows to the Businesses sheet
- Swap WhatsApp for Telegram notifications
- Add a Notion database as secondary data store
- Connect to HubSpot CRM to log review data against contacts
- Add a "Response Sent" tracker column for closed-loop management
- Extend AI categories for specific industries (e.g., "Wait Time" for restaurants, "Bedside Manner" for clinics)

---

## 📬 Hire Me

I build custom review monitoring and reputation intelligence systems for agencies and multi-location businesses.

- 🔗 **Upwork:** [*Dhanraj Sharma*](https://www.upwork.com/freelancers/~010e4c7ac19e0fdda1?mp_source=share)
- 🔗 **Contra:** [*Dhanraj Sharma*](https://contra.com/dhanraj_sharma_rgam8kpb?referralExperimentNid=DEFAULT_REFERRAL_PROGRAM&referrerUsername=dhanraj_sharma_rgam8kpb)
- 💼 **LinkedIn:** [*Dhanraj Sharma*](https://www.linkedin.com/in/dhanraj-sharma-nddimension/)
- 📧 **Email:** *hinatashoyo101824@gmail.com*

---

## 📄 License

This project is released under a proprietary license.

The repository is provided for portfolio and evaluation purposes only. Commercial use, redistribution, resale, and client deployment are prohibited without explicit written permission from the author.

---

*Built by Dhanraj Sharma — AI Automation Specialist*
*Ex-ISRO Research Intern | B.Tech AI/ML, Gujarat*
