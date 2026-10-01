# ShopBot: AI shopping assistant in a WhatsApp-style chat

An MBA startup **prototype and simulation**. It answers one question:

> Can an AI shopping agent let a consumer search across multiple e-commerce platforms, compare prices, choose a seller, and complete a simulated purchase entirely through conversation?

Nothing here connects to Amazon, Flipkart, WhatsApp, payment gateways or any real service. All prices, stock, ratings and delivery times are **simulated demo data**.

## A. How to run it

Option 1 (no install): double-click `index.html`. It runs in any modern browser, offline.

Option 2 (local server):

```bash
node server.js          # then open http://localhost:3000
```

Run the automated journey tests (44 checks, no dependencies):

```bash
node tests/journey.test.js
```

Requires Node 18+ only for the server and tests. The app itself is plain HTML, CSS and JavaScript.

## B. Folder structure

```
shopbot/
  index.html            Page shell: chat window, info drawer (About / Business model / Demo guide)
  css/styles.css        Design system and layout (responsive)
  js/
    catalog.js          MOCK DATABASE: platforms, categories, 37 products, 156 listings. Edit this file.
    store.js            Builds full listing records (discount, delivery date, fees, return period) + text utilities
    engine.js           The agent: language understanding, conversation memory, checkout, payment, orders
    ui.js               Renders agent messages as chat bubbles, cards, tables; handles clicks and demo prompts
  tests/journey.test.js Scripted end-to-end tests of the whole journey and every edge case
  server.js             Optional tiny static server
  package.json          npm start / npm test
```

## C. Main features

- WhatsApp-style chat with product cards, quick-reply buttons, checkout and payment cards, order confirmation and tracking
- 10 platforms: Amazon, Flipkart, Blinkit, Zepto, Instamart, BigBasket, Nykaa, Myntra, Croma, Reliance Digital
- 7 categories: smartphones, laptops, headphones, fashion, home appliances, groceries, beauty
- Natural-language queries: "Find iPhone 17", "phones under 50000", "cheapest MacBook", "show Amazon options", "compare the cheapest three", "buy the second one", "cancel my order", "where is my order?"
- Conversation memory: "it", "this", "the second one", "the Flipkart one" all resolve to the right listing
- No information overload: category and budget searches show at most 6 picks
- Honest comparison table that highlights the best value per row but never picks for the user
- Delivery fees are included in "cheapest" (a Rs 57 milk plus Rs 25 fee is Rs 82), which matters for quick commerce
- Simulated UPI, card and cash-on-delivery payment with a processing state, transaction ID and order ID
- Order tracking with a "Simulate next update" button, and order cancellation (allowed before shipping)
- Edge cases: unknown product, nothing under budget (shows closest options), ambiguous query, unavailable item, unknown order ID
- Trust labels: "Demo data" pill, "Simulated" tags on every checkout and payment card, and a footer disclaimer

## D. Demo script (about 5 minutes)

Use the suggested prompts under the chat, or type the lines below.

| Step | You say | What it shows |
|---|---|---|
| 1 | (open the page) | Greeting, one-line positioning, quick buttons |
| 2 | **Find me Sony WH-1000XM6** | 4 platforms side by side. ShopBot says Flipkart is cheapest while Amazon and Croma are fastest |
| 3 | Click **Cheapest** | "Flipkart is the cheapest at 33,499" with Buy This / Compare Again |
| 4 | Click **Compare Again** | Comparison table. Point out that ShopBot does not choose for the user |
| 5 | Type **Buy this** after sorting, or click **Choose Flipkart** | Order summary, delivery address, Confirm / Change Address |
| 6 | **Confirm Order**, then **UPI**, type any `name@bank`, **Pay** | Simulated payment, transaction ID, order ID `SB-<today>-1042` |
| 7 | **Where is my order?** | Tracking timeline. Click "Simulate next update" to move it forward |
| 8 | **Start fresh**, then **Find headphones under 35,000** | Budget filter, only relevant picks. Click **All sellers** on one |
| 9 | **Find iPhone** | Ambiguity handling: "Do you mean iPhone 17 or iPhone 17 Pro?" |
| 10 | **Amul butter 500g**, then **fastest** | Quick commerce: Zepto in 9 mins. Mention delivery fees are counted |
| 11 | Open **How we make money** (top bar) | User, ShopBot, Merchant. Commission model, plus a clear note that the prototype earns nothing |

Context demo: **Find AirPods Pro**, then **Which is cheapest?**, then **Buy it**. ShopBot knows "it" means the Flipkart AirPods.
Edge-case demo: **Find iPhone 99** (unknown), **iPhone 17 under 50000** (closest options), **Find iPhone 17 Pro** then **Buy the Reliance one** (unavailable).

## E. Future scope

- Real catalog through merchant and affiliate APIs, with live price and stock refresh
- A real WhatsApp Business channel and a hosted LLM for open-ended language
- Real payments through a licensed payment aggregator, and order status from merchant webhooks
- Price-drop alerts, wishlists and reorder for groceries
- Personalisation: saved addresses, brand preferences, past purchases
- Sponsored placements (clearly labelled) and merchant analytics dashboard
- Regional language and voice input

## F. Business model

`USER -> SHOPBOT -> MERCHANT`

The user pays the normal listed price. When a purchase is completed through a partner platform, that platform pays ShopBot a commission or referral fee. Possible extra streams: sponsored placement, merchant referral fees, a premium subscription (alerts and deal tracking), and merchant analytics. The prototype itself does not earn any money.

## G. Limitations of this prototype

- All data is invented and static. Prices will not match real listings.
- The "AI" is a rule-based language understander over a fixed catalog, not a language model. It handles the patterns in this README well but will not understand arbitrary requests.
- One unit per order. No cart, sizes, colours or variants.
- Payment, orders and tracking are simulated and live only in the open browser tab. Refreshing the page or **Start fresh** clears them.
- No login, accounts or real addresses. Two saved addresses plus free-text entry.
- Delivery times are fixed per listing, not calculated from the user's pin code.
- Google Fonts are loaded if online; offline it falls back to system fonts.

## Editing the mock data

Open `js/catalog.js`. To add a product, copy a block in `SB.products` and list where it is sold:

```js
{ id: 'p38', name: 'Your Product', brand: 'Brand', category: 'headphones', icon: '🎧', mrp: 9999,
  specs: 'One-line description',
  tags: ['headphone', 'headphones'],           // generic words people use for this kind of product
  aliases: ['short name', 'nickname'],         // other names people might type
  listings: [L('amazon', 7999, 4.4, 1200, '1d'), L('zepto', 8199, 4.3, 90, '9m', { stock: 'low' })] }
```

Delivery codes: `'10m'` minutes, `'2h'` hours, `'today'`, `'1d'` tomorrow, `'3d'` three days. Options: `{ stock: 'low' | 'out' }` or `{ mrp: 12000 }`. To change a platform's delivery fee or return window, edit `SB.platforms`.

## WhatsApp mode (optional)

`whatsapp/` contains a small webhook backend that runs the same engine behind the real WhatsApp Cloud API (Meta test number). Setup steps are in `WHATSAPP_SETUP.md`. Replies become WhatsApp text, reply buttons and list menus. Tests: `node tests/whatsapp.test.js`.
