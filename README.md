# Doctor microsites + WhatsApp booking assistant (Vercel)

Start with **SETUP-GUIDE.md**.

| Page | What it is |
| --- | --- |
| `/` | demo directory of the 5 doctors |
| `/#dr-reddy` ... `/#dr-misha` | standalone microsites with booking, WhatsApp CTA and QR code |
| `/#qr` | all doctor QR codes, ready to print |
| `/#chat` | test the WhatsApp assistant in the browser |

| Endpoint | Used by |
| --- | --- |
| `/api/whatsapp` | Meta WhatsApp webhook |
| `/api/chat` | the browser test chat |
| `/api/slots`, `/api/book` | website booking |
| `/api/config` | the website |

Code: `api/` endpoints, `lib/agent.js` (conversation and tools), `lib/rag.js` (knowledge search),
`lib/booking.js` (slots, no double booking), `lib/store.js` (Upstash Redis), `lib/whatsapp.js`.
Content: `public/data/doctors.json`, `public/data/faq.json`.
