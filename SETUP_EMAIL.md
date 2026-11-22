# Connecting the contact form to real email (Supabase Edge Function + Resend)

The contact form posts to a **server-side** Supabase Edge Function (`supabase/functions/send-contact`)
which validates the enquiry, rate-limits it and sends it to **shreyascoding2313@gmail.com** through
[Resend](https://resend.com). The form only shows "Message delivered" when Resend confirms it accepted
the email (returns a message id). Until you complete the steps below the form shows a clear
"email delivery isn't connected" message and a `mailto:` fallback — it never pretends to send.

## 1. Create the accounts
1. A [Supabase](https://supabase.com) project (free tier is fine).
2. A [Resend](https://resend.com) account and an API key.
   - Testing: the default sender `onboarding@resend.dev` can only deliver to the email address you
     registered Resend with. Register Resend with `shreyascoding2313@gmail.com`, or
   - Production: verify your own domain in Resend and set `CONTACT_FROM_EMAIL` (below).

## 2. Create the rate-limit table
```bash
npm i -g supabase          # or: npx supabase ...
supabase login
supabase link --project-ref YOUR-PROJECT-REF
supabase db push           # applies supabase/migrations/*.sql
```
(Or paste `supabase/migrations/20250101000000_contact_attempts.sql` into the Supabase SQL editor.)

## 3. Add the secrets (server side only)
```bash
supabase secrets set RESEND_API_KEY=re_your_key
# optional
supabase secrets set CONTACT_TO_EMAIL=shreyascoding2313@gmail.com
supabase secrets set CONTACT_FROM_EMAIL="SHREYAS TECHNOLOGIES <hello@your-verified-domain.com>"
supabase secrets set ALLOWED_ORIGINS=https://your-site.com
```

## 4. Deploy the function
```bash
supabase functions deploy send-contact --no-verify-jwt
```

## 5. Point the website at it
Set the public project URL at build time and rebuild:
```bash
echo "VITE_SUPABASE_URL=https://YOUR-PROJECT-REF.supabase.co" > .env
npm run build
```

## Protections included
| Protection | How |
| --- | --- |
| Secret safety | `RESEND_API_KEY` exists only as a Supabase secret; the browser never sees it |
| Validation | Same rules on client and server; server whitelists service/budget values |
| Spam | Hidden honeypot field + minimum 3 s fill-time check |
| Rate limiting | Hashed IP: 3/hour and 8/day; hashed email: 4/day (stored in `contact_attempts`, fails closed) |
| Reply-to | Visitor's email is set as `reply_to` |
| Injection | All user content HTML-escaped; header fields stripped of line breaks |
| Honest status | Success only when Resend returns an email id |

For stronger bot protection you can additionally add Cloudflare Turnstile and verify its token inside the function.
