# Put ShopBot on real WhatsApp (Meta test number)

Everything stays simulated (catalog, payment, orders). Only the chat channel becomes real WhatsApp.
Needs about 1 to 2 hours the first time. Menu names at Meta change now and then, so go by the wording, not exact positions.

Code for this lives in `whatsapp/` (server.js and format.js). Tests: `node tests/whatsapp.test.js`.

## Part 1. Put the code on GitHub (10 min)
1. Create a free account at github.com.
2. New repository, name `shopbot`, Public, Create.
3. Click "uploading an existing file", drag in EVERYTHING from the unzipped `shopbot` folder (index.html, css, js, whatsapp, tests, server.js, package.json, README.md, WHATSAPP_SETUP.md). Commit.
4. Check that `whatsapp/server.js` and `package.json` are visible at the top level of the repo.

## Part 2. Meta developer account and app (20 min)
1. Go to developers.facebook.com, click Get Started, log in with Facebook, verify email and phone, accept terms.
2. My Apps > Create App.
3. Choose the use case "Connect with customers through WhatsApp" (if you get an older flow, choose type "Business" and then add the WhatsApp product).
4. App name `ShopBot Demo`, your email. When asked for a business portfolio, create one (free, only a name and email needed) or pick an existing one. Create app.
5. In the left menu open WhatsApp > API Setup (also called Quickstart). Keep this tab open.
6. Write down: Phone number ID (under the test number), WhatsApp Business Account ID, and the temporary access token.

## Part 3. Allow your phones to chat with the test number (5 min)
1. On the API Setup page, find "To" and click Manage phone number list.
2. Add each teammate's and your professor's WhatsApp number with country code (India: +91...). Meta sends a code to each phone on WhatsApp, enter it to verify. Up to 5 numbers.
3. Only these verified numbers can message the test number.

## Part 4. Deploy the backend on Render (20 min)
1. Go to render.com and sign up with GitHub.
2. New > Web Service > connect your `shopbot` repo.
3. Settings: Name `shopbot-wa`, Runtime Node, Build Command `npm install`, Start Command `node whatsapp/server.js`, Instance type Free.
4. Under Environment Variables add:
   - VERIFY_TOKEN = any secret you invent, e.g. shopbot-xlri-2026
   - WHATSAPP_TOKEN = the temporary access token from Part 2
   - PHONE_NUMBER_ID = the Phone Number ID from Part 2
   - APP_SECRET = (optional) Meta app dashboard > App settings > Basic > App secret > Show
5. Create Web Service. Wait for "Live". Copy your URL, e.g. https://shopbot-wa.onrender.com
6. Open that URL in a browser. You should see "ShopBot WhatsApp backend is running."

## Part 5. Connect Meta to your server (10 min)
1. In the Meta app dashboard: WhatsApp > Configuration.
2. Webhook > Edit.
3. Callback URL: your Render URL + /webhook, e.g. https://shopbot-wa.onrender.com/webhook
4. Verify token: exactly the VERIFY_TOKEN you set on Render.
5. Click Verify and save. If it fails, see Troubleshooting.
6. Under Webhook fields click Manage and subscribe to `messages`.

## Part 6. Test (5 min)
1. From a verified phone, open WhatsApp and message the test number. Its number is shown on the API Setup page.
2. Send: Find me Sony WH-1000XM6
3. You should get the comparison and a list menu. Tap Cheapest, Buy This, Confirm Order, UPI, Use demo UPI ID.
4. Send: where is my order?

## Part 7. Make the token last (15 min, do before the demo)
The temporary token dies after about 24 hours.
1. Go to business.facebook.com > Business settings.
2. Users > System users > Add. Name `shopbot-bot`, role Admin.
3. Select it > Add assets > Apps > select your app > Full control > Save.
4. Also Add assets > WhatsApp accounts > select your test account > Full control > Save.
5. Generate new token > pick your app > expiry Never > tick whatsapp_business_messaging and whatsapp_business_management > Generate. Copy it.
6. Render > your service > Environment > replace WHATSAPP_TOKEN with this token > Save (it redeploys).

## Before presenting
- Render free services sleep after about 15 minutes idle and take up to a minute to wake. Open your Render URL, then send a test message about 10 minutes before you present.
- Orders and chats live in server memory and reset when the server restarts or redeploys.
- Keep the browser version open as a backup.

## Troubleshooting
- Verify and save fails: VERIFY_TOKEN differs from Meta (check spaces), the URL lacks /webhook, or Render is still starting. Open the URL in a browser first.
- No reply on WhatsApp: check Render > Logs. Make sure the `messages` field is subscribed and your number is on the allowed list.
- Logs say "Send failed 401" or "190": the token expired or was copied wrongly. Do Part 7.
- Logs say "Send failed 400" with a recipient error: the phone isn't in the allowed list.
- You only get replies after YOU message first. WhatsApp allows free-form replies only within 24 hours of the user's last message.
- Test locally without Meta: `DRY_RUN=1 VERIFY_TOKEN=x node whatsapp/server.js`, then POST a sample webhook. Replies print to the console.
