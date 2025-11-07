# 🤖 AI Automation Content Workflow – n8n Implementation

## 🧭 Overview
This workflow automates content research, prompt generation, blog/video creation, and content submission for review — all powered by AI and APIs.

It is composed of **four agents**, each responsible for a specific phase of the automation pipeline.

---

## ⚙️ AGENT 1: Content Research Agent

**Goal:** Discover trending topics in the “AI Automation” niche using Google Trends and YouTube.

### 🔹 Tools & Integrations
- **Google Trends API** (via HTTP Request)
- **YouTube Data API v3** (via HTTP Request)
- **Schedule Trigger**

### 🧱 n8n Steps
1. **Trigger Node (Schedule)**
   - Type: `Cron`
   - Schedule: Weekly, Monday 09:00
2. **HTTP Request: Google Trends**
   - **URL:** `https://trends.google.com/trends/api/explore`
   - **Method:** `GET`
   - **Query Params:**
     ```
     hl=en
     geo=US
     q=AI Automation
     ```
   - Extract top trending keywords.
3. **HTTP Request: YouTube Data API**
   - **URL:** `https://www.googleapis.com/youtube/v3/search`
   - **Query Params:**
     ```
     part=snippet
     q=AI Automation
     type=video
     order=viewCount
     maxResults=10
     key={{YOUR_YOUTUBE_API_KEY}}
     ```
   - Extract video titles.
4. **Merge Node**
   - Combine Google Trends + YouTube outputs.
5. **Remove Duplicates Node**
   - Clean topics list.
6. **Output**
   - Send unique topics array to **Prompt Agent**.

---

## ⚙️ AGENT 2: Prompt Agent

**Goal:** Generate creative content prompts for each trending topic.

### 🔹 Tools & Integrations
- **OpenAI Node** *(or Gemini Node)*
- **Split In Batches Node**

### 🧱 n8n Steps
1. **Receive Input** from Agent 1.
2. **Split In Batches Node** – iterate over topics.
3. **OpenAI Node**
   - **Model:** `gpt-4-turbo`
   - **Prompt:**
     ```
     You are a creative content strategist.
     Generate 3 engaging content ideas for the topic: "{{topic}}".
     Output in JSON format:
     {
       "topic": "{{topic}}",
       "prompts": ["Prompt 1", "Prompt 2", "Prompt 3"]
     }
     ```
4. **Output**
   - Forward each topic + generated prompts to **Content Creator Agent**.

---

## ⚙️ AGENT 3: Content Creator Agent

**Goal:** Generate blog posts and videos from the prompts.

### 🔹 Tools & Integrations
- **OpenAI Node** (for text generation)
- **HTTP Request Node** (Google VEO 3 API)
- **Merge Node**

### 🧱 n8n Steps
1. **Receive topic + prompt** from Agent 2.
2. **OpenAI Node (Blog Post Generation)**
   - **Prompt:**
     ```
     Write a 600-word blog post on "{{topic}}" using the idea "{{prompt}}".
     Include intro, 3 key insights, and conclusion.
     ```
   - Output: `blog_content`
3. **HTTP Request Node (Video Generation via Google VEO 3 API)**
   - **Method:** POST  
   - **URL:** `https://video.googleapis.com/v1beta/veo/generate`
   - **Headers:**
     ```
     Authorization: Bearer {{YOUR_ACCESS_TOKEN}}
     Content-Type: application/json
     ```
   - **Body:**
     ```json
     {
       "prompt": "{{prompt}}",
       "topic": "{{topic}}"
     }
     ```
   - Output: `video_link`
4. **Merge Node**
   - Combine blog + video link:
     ```json
     {
       "topic": "AI Workflow Tools",
       "prompt": "How automation transforms productivity",
       "blog_content": "Full generated text...",
       "video_link": "https://veo.google.com/xyz123"
     }
     ```
5. **Output**
   - Pass to **Content Submission Agent**.

---

## ⚙️ AGENT 4: Content Submission & Review Agent

**Goal:** Submit generated content to Google Sheets for human review.

### 🔹 Tools & Integrations
- **Google Sheets Node**
- **Add Timestamp Node**
- **Optional:** Slack or Email Notification

### 🧱 n8n Steps
1. **Add Timestamp Node**
   - `{{ $now }}`
2. **Google Sheets Node**
   - **Action:** Append Row
   - **Columns:**
     | Timestamp | Topic | Prompt | Blog Content | Video Link | Status |
   - **Default Status:** “Pending Review”
3. **Optional Slack/Gmail Node**
   - Notify reviewer:
     ```
     New content submitted for review!
     Topic: {{topic}}
     Status: Pending Review
     ```

---

## ⚠️ ERROR HANDLING & LOGGING

| Step | Handling |
|------|-----------|
| API Calls | Continue on fail, log errors to separate Google Sheet tab. |
| OpenAI Node | Check for empty response, send alert email. |
| Merge | Validate item counts. |
| Sheets Append | Retry 3x before alert. |

---

## 🧩 WORKFLOW MAP

```
Schedule → Google Trends + YouTube API
           ↓
        Merge & Dedup
           ↓
        Prompt Agent (OpenAI)
           ↓
     Content Creator (Blog + Video)
           ↓
     Google Sheet Append → Slack Notification
```

---

## 🧾 SAMPLE GOOGLE SHEET ENTRY

| Timestamp | Topic | Prompt | Blog Content | Video Link | Status |
|------------|--------|---------|---------------|--------------|----------|
| 2025-11-07 09:00 | AI Workflow Tools | How automation transforms productivity | [Generated blog text] | https://veo.google.com/abc123 | Pending Review |

---

## 📁 SUBMISSION PACKAGE CHECKLIST

✅ `ai_automation_content_workflow.json` (workflow export)  
✅ Screenshots of each agent (nodes & configurations)  
✅ Sample Google Sheet with a row of generated content  
✅ This documentation (`README.md`)  
✅ (Bonus) Notification alert screenshot

---

## 💡 FUTURE IMPROVEMENTS
- Add SEO keyword scoring.
- Auto-publish to WordPress/YouTube after approval.
- Generate social post captions automatically.
