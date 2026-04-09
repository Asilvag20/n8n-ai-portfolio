# Demo Guide — Flow 01: AI Lead Qualification Agent

Video target: **60 seconds** · Format: GIF or MP4 · Audience: potential clients / recruiters

---

## What to Have Open Before Recording

Arrange these windows on screen simultaneously — use a 2-column layout:

| Left side | Right side |
|-----------|------------|
| `forms/lead-form.html` (browser) | n8n canvas → Executions tab |
| — | Airtable "Leads" table (browser tab) |
| — | Gmail inbox (browser tab) |

**Recommended layout:** browser left half, n8n right half. Switch tabs on the right to show Airtable and Gmail at the right moments.

---

## 60-Second Script

### 0:00 – 0:08 | Show the problem (context)
- Screen: n8n canvas with all nodes visible
- Say (or caption): *"Sales teams waste hours qualifying leads manually. This agent does it in seconds."*
- Slowly pan across the flow from left to right so viewers see the full pipeline

### 0:08 – 0:18 | Submit a lead via the form
- Switch to `lead-form.html`
- Fill out the form with the test data below (type slowly, it reads better on video)
- Click **"Solicitar consultoría gratuita"**
- Show the loading spinner briefly

### 0:18 – 0:35 | Watch n8n execute in real time
- Switch to n8n → **Executions** tab
- Watch the execution appear and nodes light up green one by one
- Pause on the **AI Lead Qualifier** node — it takes ~5–10s while Ollama thinks
- Show the execution completing with all nodes green

### 0:35 – 0:45 | Show Airtable record created
- Switch to Airtable tab
- Show the new row: name, company, **Score: 87**, **Tier: HOT**
- Scroll right to show Key Insights and Personalized Opener columns

### 0:45 – 0:55 | Show email notification
- Switch to Gmail inbox
- Show the email subject: *"🔥 [URGENT] HOT Lead: Carlos Mendoza - TechCorp Colombia"*
- Open it — show the HTML layout with score badge and AI insights

### 0:55 – 1:00 | End card
- Back to n8n canvas (full flow view)
- Caption: *"Built with n8n + Ollama (local AI) + Airtable · Zero API cost · < 15s end-to-end"*

---

## Test Data for the Demo

Copy-paste into the form or use the curl command:

**Form values:**
```
Name:     Carlos Mendoza
Email:    carlos.mendoza@techcorp.co
Company:  TechCorp Colombia
Phone:    +57 310 555 0000
Source:   Website
Message:  Estamos buscando automatizar nuestro proceso de ventas.
          Somos una empresa de 50 personas y necesitamos una solución
          urgente para este trimestre. Presupuesto disponible de
          $5000 USD mensuales.
```

**curl command (for terminal demo):**
```bash
curl -X POST http://localhost:5678/webhook/new-lead \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Carlos Mendoza",
    "email": "carlos.mendoza@techcorp.co",
    "company": "TechCorp Colombia",
    "message": "Estamos buscando automatizar nuestro proceso de ventas. Somos una empresa de 50 personas y necesitamos una solución urgente este trimestre. Presupuesto: $5000 USD/mes.",
    "source": "Website"
  }'
```

Expected result: **Score ~85–92, Tier: HOT**

---

## Tips for a Clean Recording

- **Hide browser bookmarks bar** — less visual noise (Ctrl+Shift+B)
- **Use full screen** for each window — no distractions
- **Zoom n8n canvas** to 80% so all nodes are visible without scrolling
- **Pre-fill Airtable** with 2–3 fake older leads so the new one appears as a fresh addition, not alone
- **Clear Gmail inbox** of unrelated emails before recording
- **Run one test execution first** to warm up Ollama — first call is slower

---

## Free Recording Tools for Windows

| Tool | Best for | Link |
|------|----------|------|
| **ShareX** | GIF + MP4, free, no watermark | shareX.github.io |
| **ScreenToGif** | Lightweight GIF recorder | screentogif.com |
| **OBS Studio** | Full MP4 video, pro quality | obsproject.com |
| **Xbox Game Bar** | Built into Windows 11 (Win+G) | No install needed |

**Recommended for portfolio:** Record with OBS (MP4) → upload to YouTube as unlisted → embed the link in the README. Then use ScreenToGif to create a short GIF for the GitHub README preview.

---

## GIF Optimization

If your GIF is too large for GitHub (>10MB):
```bash
# Using ffmpeg (free)
ffmpeg -i demo.gif -vf "fps=10,scale=800:-1" -loop 0 demo-optimized.gif
```

Or upload to [ezgif.com/optimize](https://ezgif.com/optimize) and reduce to 10fps + 70% quality.
