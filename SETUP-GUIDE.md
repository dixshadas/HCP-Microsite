# Setup guide: doctor microsites + WhatsApp booking assistant on Vercel

This project is **Project 1**: 5 doctor microsites, online booking, printable QR codes, and a WhatsApp
AI assistant that answers questions and books, reschedules and cancels appointments.
The Erecta patient bot is a separate project with its own guide.

You need free accounts on **GitHub**, **Vercel**, an **AI provider** (OpenAI or Google Gemini) and,
for WhatsApp, **Meta for Developers**.

> **Vercel plan:** the free Hobby plan is for personal, non-commercial use only. A pilot for a company
> needs the Pro plan. You can test on Hobby, but move to Pro before real patients use it.

---

## Step 1: Put the project on GitHub (5 minutes)

1. Sign in at https://github.com, click **New repository**, name it `doctor-microsites`, choose **Private**, then **Create repository**.
2. Click **uploading an existing file**.
3. Unzip `doctor-microsites-vercel.zip`, open the folder and drag **everything inside it** onto the page
   (`api`, `lib`, `public`, `tools`, `package.json`, `vercel.json` and the rest).
4. Click **Commit changes**.

## Step 2: Deploy on Vercel (3 minutes)

1. Sign in at https://vercel.com with your GitHub account.
2. Click **Add New... > Project**, pick `doctor-microsites`, then **Import**.
3. **Framework Preset:** choose **Other**. Leave everything else as it is and click **Deploy**.
4. Open the link Vercel gives you, for example `https://doctor-microsites.vercel.app`. You will see the doctor directory.

## Step 3: Connect storage for bookings (3 minutes)

Bookings, conversation memory and duplicate-message checks are kept in Redis.

1. In your Vercel project, open the **Storage** tab, then **Create Database** (or **Browse Marketplace**).
2. Choose **Upstash > Upstash for Redis**, pick the free plan and a region close to your patients (e.g. Mumbai), and create it.
3. Connect it to this project. Vercel adds `KV_REST_API_URL` and `KV_REST_API_TOKEN` for you.
4. Go to **Deployments**, open the latest one, click **... > Redeploy**.

Until this is done the site shows "Bookings are not being saved yet".

## Step 4: Turn on the AI (5 minutes)

1. Create an API key: OpenAI at https://platform.openai.com/api-keys, or Gemini at https://aistudio.google.com/apikey.
2. In Vercel: **Settings > Environment Variables**, add:

| Name | Value |
| --- | --- |
| `LLM_API_KEY` | your key |
| `LLM_MODEL` | model name; default `gpt-6-luna` (OpenAI's low-cost model). For Gemini use a current Flash model |
| `LLM_BASE_URL` | Gemini only: `https://generativelanguage.googleapis.com/v1beta/openai` |
| `ORG_NAME` | how the assistant names your organisation |
| `ASSISTANT_NAME` | the assistant's first name (default `Asha`) |

3. **Redeploy**, then open `https://<your-site>.vercel.app/#chat` and chat. This is the same assistant
   WhatsApp patients will reach.

## Step 5: Connect a WhatsApp number (20 minutes)

1. https://developers.facebook.com/apps > **Create app** (Business type, or the WhatsApp use case), linked to your Meta Business account.
2. **WhatsApp > API Setup**: note the **test phone number**, its **Phone number ID**, and the **temporary access token**.
   Add and verify your own WhatsApp number under **To** (the test number only messages verified numbers).
3. In Vercel add:

| Name | Value |
| --- | --- |
| `WHATSAPP_TOKEN` | the access token |
| `WHATSAPP_PHONE_NUMBER_ID` | the Phone number ID |
| `WHATSAPP_VERIFY_TOKEN` | a password you make up |
| `WHATSAPP_APP_SECRET` | Meta app **App settings > Basic > App secret** |
| `WHATSAPP_DISPLAY_NUMBER` | the number patients write to, digits only with country code (e.g. `15551234567`). All QR codes and buttons use it |

4. **Redeploy.**
5. In Meta: **WhatsApp > Configuration > Webhook > Edit**
   - Callback URL: `https://<your-site>.vercel.app/api/whatsapp`
   - Verify token: the same as `WHATSAPP_VERIFY_TOKEN`
   - **Verify and save**, then subscribe to the **messages** field.
6. Send "Hi" from your phone. You'll see blue ticks, "typing..." and a reply within seconds.

**Going live:** register your real number in WhatsApp Manager, complete business verification, create a
permanent token (Business Settings > System users, permission `whatsapp_business_messaging`), update the
variables and redeploy.

**Confirmations for website bookings:** sent straight away if the patient messaged in the last 24 hours.
Otherwise WhatsApp needs an approved template: set `WHATSAPP_CONFIRM_TEMPLATE` (4 variables: name, doctor,
time and clinic, ref) and optionally `WHATSAPP_DOCTOR_ALERT_TEMPLATE`.

---

## Editing content

| File | What it holds |
| --- | --- |
| `public/data/doctors.json` | doctors, qualifications, awards, publications, associations, clinics, days, hours, fees, alert numbers |
| `public/data/faq.json` | payment, walk-ins, cancellation, what to bring, emergencies |

Edit them on GitHub (open the file, click the pencil, commit); Vercel redeploys by itself.
**All current content is sample data.** Replace it before patients use the site.

Changing the page design: edit `tools/app.js` or `tools/index.template.html`, run `python3 tools/build.py`,
upload the new `public/index.html`.

## Good to know

- Meta gets an instant "received" and the assistant replies in the background, so slow AI replies never cause duplicate messages.
- Without an AI key the assistant still answers with simple automatic replies (doctor list, timings, free slots, booking link).
- Turn off the public test chat with `ENABLE_WEB_CHAT=false`.
- Add a privacy notice and consent line before launch (India DPDP Act, Kenya Data Protection Act, US rules).
