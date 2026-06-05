# 🔧 ReviewRadar — Complete Setup Guide

Step-by-step instructions to deploy the full ReviewRadar system from scratch.

**Estimated setup time: 60–90 minutes**

---

## Prerequisites — Accounts to Create First

Before importing any workflow, create accounts on these platforms:

| Platform | Purpose | Cost |
|----------|---------|------|
| n8n.io | Automation engine | ~$20/month (cloud) or free (self-hosted) |
| serpapi.com | Google review fetching | 100 free searches/month |
| aistudio.google.com | Gemini AI API key | Free (1,500 req/day) |
| Google account | Sheets + Gmail + Looker | Free |
| Slack | Alert notifications | Free |
| twilio.com | WhatsApp alerts | Free trial available |

---

## Step 1 — Set Up Google Sheets

Create a new spreadsheet named `ReviewRadar-Sheets`.

Create **4 tabs** with these exact column headers:

### Tab 1: `Businesses`
```
Business Name | SERP Data ID | Owner Email
```
Add your businesses here. Example row:
```
Cozy Restaurant | 0x3be06b7b6ef0f68d:0x... | owner@restaurant.com
```

> **How to find SERP Data ID:**
> 1. Go to Google Maps → search your business
> 2. Right-click the URL → copy it
> 3. Go to serpapi.com/playground → paste URL → it extracts the data_id for you
> Alternatively: use the SERP API playground with `engine=google_maps` and search the business name — the result contains the `data_id`

### Tab 2: `All Reviews`
```
Review ID | Business Name | Reviewer Name | Rating | Review Text | Date | Sentiment | Category | Urgency | Draft Response | Processed At
```

### Tab 3: `Testimonials`
```
Business Name | Reviewer Name | Rating | Review Text | Date | Category
```

### Tab 4: `Weekly Summary`
```
Business Name | Week Of | Avg Rating | Top Complaints | Top Praise | Recommendation
```

---

## Step 2 — Get Your API Keys

### SERPAPI Key
1. Go to serpapi.com → Sign up (free)
2. Dashboard → Your API Key → Copy it
3. Free tier: 100 searches/month (enough for testing + small deployments)

### Google Gemini API Key
1. Go to aistudio.google.com
2. Click **Get API Key** → Create API Key
3. Copy the key — it starts with `AIza...`
4. Free tier: 1,500 requests/day on Gemini 1.5 Flash

### Twilio WhatsApp Setup
1. Go to twilio.com → Sign up (free trial gives ~$15 credit)
2. Console → Messaging → Try WhatsApp (Sandbox)
3. Note your: Account SID, Auth Token, and WhatsApp From number (`whatsapp:+14155238886` on trial)
4. Send the join code to the sandbox number from your personal WhatsApp
5. Add your WhatsApp number as the "To" number in n8n

### Slack Webhook
1. Go to api.slack.com/apps → Create New App → From Scratch
2. Add **Incoming Webhooks** → Activate → Add to Workspace
3. Select the channel for review alerts (e.g., `#review-alerts`)
4. Copy the Webhook URL (starts with `https://hooks.slack.com/services/...`)

---

## Step 3 — Set Up n8n Credentials

In your n8n dashboard → **Credentials** → create these:

### Google OAuth (Sheets + Gmail)
1. Add Credential → Google Sheets OAuth2 API
2. Follow the OAuth flow → allow Sheets + Gmail + Drive scopes
3. Name it: `Google Account`

### Google Gemini
1. Add Credential → Google PaLM API (this is what n8n calls the Gemini credential)
2. Paste your Gemini API key
3. Name it: `Google Gemini API`

### SERPAPI (via HTTP Request — no native node needed)
- Used directly inside an HTTP Request node as a query parameter
- No separate credential needed — just paste the key in the node

### Slack
1. Add Credential → Slack API
2. Paste your Slack Webhook URL
3. Name it: `Slack Account`

### Twilio
1. Add Credential → Twilio API
2. Account SID + Auth Token from your Twilio console
3. Name it: `Twilio Account`

---

## Step 4 — Import the Workflows

1. n8n Dashboard → **New Workflow** → three-dot menu → **Import from File**
2. Upload `ReviewRadar_Watcher.json`
3. Repeat for `ReviewRadar_Weekly_Report.json`

---

## Step 5 — Configure the Watcher Workflow

Open `ReviewRadar - Watcher`.

### 5a — Google Sheets nodes
For every Sheets node:
- Click node → Credential → select `Google Account`
- Select `ReviewRadar-Sheets` spreadsheet
- Select the correct tab (Businesses / All Reviews / Testimonials)

### 5b — SERPAPI HTTP Request node
Click the HTTP Request node:
- URL: `https://serpapi.com/search`
- Method: GET
- Query Parameters:
  ```
  engine = google_maps_reviews
  data_id = {{ $json["SERP Data ID"] }}
  api_key = YOUR_SERPAPI_KEY_HERE
  ```

### 5c — Gemini AI node
- Click the Google Gemini Chat Model node → select `Google Gemini API` credential
- Model: `models/gemini-1.5-flash` (confirm this is selected, not a non-existent model name)

> **Important:** The weekly report workflow had `gemini-3.1-flash-lite` in the model field — this model name doesn't exist. Change it to `models/gemini-1.5-flash` in both workflows.

### 5d — Gmail alert node
- Connect your Gmail credential
- Update `sendTo` to the business owner's email (or use `{{ $json["Owner Email"] }}` from the Businesses sheet)

### 5e — Slack node
- Connect your Slack credential
- Set the channel to your `#review-alerts` channel
- Message template:
  ```
  🚨 *Urgent Review Alert*
  *Business:* {{ $json["Business Name"] }}
  *Reviewer:* {{ $json["Reviewer Name"] }}
  *Rating:* {{ $json["Rating"] }}/5
  *Category:* {{ $json["Category"] }}
  *Urgency:* {{ $json["Urgency"] }}/5
  *Review:* {{ $json["Review Text"] }}
  *Draft Response:* {{ $json["Draft Response"] }}
  ```

### 5f — Twilio WhatsApp node
- Connect your Twilio credential
- From: `whatsapp:+14155238886` (Twilio sandbox number)
- To: `whatsapp:+91XXXXXXXXXX` (your number — must have joined the sandbox)
- Message: Same content as Slack message above

---

## Step 6 — Configure the Weekly Report Workflow

Open `ReviewRadar - Weekly Report`.

1. **Collect "All Reviews" node:** Connect Google credential → select All Reviews tab
2. **Google Gemini Chat Model node:** Connect Gemini credential → change model to `models/gemini-1.5-flash`
3. **Send a message (Gmail) node:** Connect Gmail credential → update `sendTo`
4. **Weekly Report Sheet node:** Connect Google credential → select Weekly Summary tab

---

## Step 7 — Set Up the Looker Studio Dashboard

1. Go to lookerstudio.google.com → **Create** → **Report**
2. **Add Data** → Google Sheets → select `ReviewRadar-Sheets` → `All Reviews` tab
3. Build Page 1 (Executive Dashboard):
   - Add **Scorecard** chart → metric: `Record Count` → label: "Total Reviews"
   - Add **Scorecard** → metric: `Rating` (Average) → label: "Average Rating"
   - Add **Bar Chart** → dimension: `Business Name` → metric: `Record Count` → title: "Reviews by Business"
   - Add **Bar Chart** → dimension: `Business Name` → metric: `Rating` (Average) → title: "Average Rating by Business"
   - Add **Time Series** → dimension: `Date` → metric: `Record Count` → breakdown: `Business Name`
   - Add **Dropdown Filter** → field: `Business Name`

4. Build Page 2 (Customer Insights):
   - Add **Pie Chart** → dimension: `Sentiment` → metric: `Record Count`
   - Add **Bar Chart** → dimension: `Category` → metric: `Record Count` → title: "Top Review Categories"
   - Add **Table** → dimensions: Business Name, Reviewer Name, Rating, Sentiment, Category, Date, Review Text → sort by Date descending

5. Build Page 3 (AI Executive Reports):
   - **Add Data** → select `Weekly Summary` tab
   - Add **Scorecard** → metric: `Avg Rating` (Average)
   - Add **Table** → Business Name + Top Complaints
   - Add **Table** → Business Name + Top Praise
   - Add **Table** → Business Name + Recommendation

6. Click **Share** → **Manage Access** → set to "Anyone with the link can view"
7. Copy the link — add it to your README and portfolio

---

## Step 8 — Test the Full System

Run these tests in order:

### Test 1 — Watcher with real data
1. Add 2 businesses to the Businesses sheet with their SERP Data IDs
2. Manually run the Watcher workflow
3. Check:
   - [ ] New reviews appear in All Reviews sheet with all columns filled
   - [ ] Urgent reviews (low rating) trigger Gmail + Slack + WhatsApp
   - [ ] 5-star reviews appear in Testimonials sheet

### Test 2 — Deduplication
1. Run the Watcher workflow twice
2. Check: no duplicate rows in All Reviews sheet

### Test 3 — Weekly Report
1. Manually trigger the Weekly Report workflow
2. Check:
   - [ ] Separate email received per business
   - [ ] Email contains top complaints, praise, and recommendation specific to that business
   - [ ] Row added to Weekly Summary sheet

### Test 4 — Dashboard
1. Open your Looker Studio dashboard
2. Check: all charts populate with data from your sheets
3. Test the Business Name filter — verify it filters all charts simultaneously

---

## Common Issues & Fixes

**Problem:** Gemini returns JSON wrapped in markdown backticks
**Fix:** The AI Output Cleaner Code node handles this automatically:
```javascript
const raw = $input.first().json.text || "";
const clean = raw.replace(/```json|```/g, "").trim();
return [{ json: JSON.parse(clean) }];
```

**Problem:** SERPAPI returns no reviews for a business
**Fix:** Verify the Data ID is correct. Test it directly at: `https://serpapi.com/search?engine=google_maps_reviews&data_id=YOUR_ID&api_key=YOUR_KEY`

**Problem:** WhatsApp messages not delivering
**Fix:** Your number must have sent the join code to the Twilio sandbox. The join code is in your Twilio console → Try WhatsApp.

**Problem:** Looker Studio shows "No data"
**Fix:** Make sure your Google Sheets have at least one row of data. Looker Studio requires the header row + at least one data row to detect schema.

**Problem:** Gemini model error in n8n
**Fix:** Change the model name to `models/gemini-1.5-flash`. The model `gemini-3.1-flash-lite` used during development does not exist — use the correct model string.

---

## You're Live 🎉

Your ReviewRadar system is now running. It will:
- Fetch new reviews on schedule
- Alert you instantly for urgent feedback
- Deliver a weekly intelligence report every Monday
- Show live reputation trends on your dashboard

For customization or questions, reach out via the links in [README.md](./README.md).
