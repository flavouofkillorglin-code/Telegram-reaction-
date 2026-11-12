# Telegram Channel Reaction Bot

**Python 3.10+ · python-telegram-bot 22.8 · SQLite**

A Telegram bot with force-sub membership checks, admin-verified channel linking, and one primary-bot reaction on each **new** post. Optional **Multi-Bot Premium** unlocks up to **five additional, owner-controlled bots** per linked channel for **100 Stars / 30 days (one-time)** or via owner manual grant. Inline-keyboard controls, SQLite persistence, and a verification animation are included.

**Minimal release:** All runtime code, SQLite storage, read-only connectivity check, and the flying-emoji GIF are merged into **one `bot.py`**. Other files are only `requirements.txt`, this README, and `.gitignore` (prevents accidentally committing secrets/database files). No extra Python script or asset folder is needed.

> **Important limit:** Telegram limits each bot to **up to one reaction per message** [1](https://core.telegram.org/bots/api). Five authorized child bots can each *try* one reaction of their own when they are channel admins; these are **bot reactions, not real members or guaranteed engagement**. Do not claim otherwise to buyers. One "Join All" button also **cannot automatically join** users to channels: users join manually and press **Check Membership**.

> **SECURITY:** A bot token was pasted into this chat. Treat it as compromised: revoke it using @BotFather (`/revoke`), generate a new one, and put the new token **only in your local/server `.env` or secret manager**. Never paste primary or child-bot tokens into a chat. This project does not contain or use that shared token.

## Quick start

1. Create a bot using Telegram **@BotFather** and obtain its token. Find your **numeric** Telegram user ID (for example using @userinfobot). **Never paste the token into public chats/repositories.**
2. In the project directory:

   ```bash
   python -m venv .venv
   source .venv/bin/activate          # Windows: .venv\Scripts\activate
   pip install -r requirements.txt
   ```

3. Create a `.env` file **on your own server/computer, beside `bot.py`**, using this template (replace placeholders **privately**):

   ```dotenv
   BOT_TOKEN=REPLACE_WITH_YOUR_NEW_PRIVATE_TOKEN
   OWNER_ID=123456789
   CHILD_BOT_TOKENS=
   DB_PATH=bot.sqlite3
   DEFAULT_EMOJI=👍
   CELEBRATION_EFFECT_ID=5046509860389126442
   ```

   Set the **newly rotated** `BOT_TOKEN` and your numeric `OWNER_ID`. Optionally choose a standard `DEFAULT_EMOJI`. For paid Premium, create **five other bots you own** in @BotFather and set `CHILD_BOT_TOKENS=tokenA,tokenB,tokenC,tokenD,tokenE` **locally**. Blank means the free bot runs but **Stars checkout and Multi React stay disabled**. `.env` is git-ignored, but **this shared workspace may persist files**: do not create a token-filled `.env` here. Keep all real secrets on **your own server/local machine or in a secret manager**. Never bundle `.env` in a ZIP or send tokens in chat.
4. **After rotating the leaked token**, run `python bot.py --selfcheck` locally: it makes read-only Telegram `getMe`/admin checks using your new local `.env`, reports public bot usernames and admin readiness, and **never sends a reaction or charges Stars**. This agent has **not run** that live check because you must revoke the shared token first. Then run `python bot.py` on a VPS/server for continuous reactions. SQLite is at `DB_PATH` (default `bot.sqlite3` next to `bot.py`). Back it up. Run **one polling instance per primary token**; stop any existing webhook. Child bots are **API clients only**; do not run pollers for them with this project.
5. Before sharing the bot, set owner-required channels: add the bot as **admin** to each required channel, then DM the bot `/addforce @publicchannel`. For a private required channel, use `/addforce -1001234567890 https://t.me/+your_invite_code`. Use `/forces` to inspect/remove requirements. With no required channels configured, `/start` immediately shows the dashboard.
6. A user sends `/start`, opens every required channel's join button, **joins manually**, then taps **Join All — Check Membership**. When verified, the dashboard is shown and the bot sends one flying-emoji GIF with a native Telegram message effect if accepted. The bot must **remain admin** in every required channel for reliable `getChatMember` checks [1](https://core.telegram.org/bots/api).
7. User taps **Set Channel** (or `/setchannel`), makes the **main** bot an **admin** of their target channel, enables reactions, and **forwards an existing channel post into the main bot's private chat**. The main bot verifies origin and admin access, links the channel and tries a test reaction. Future new posts receive the main bot's one reaction.
8. Premium users use `/childbots` to see the owner-controlled bots' **public usernames** (not tokens). They manually add **all five** as admins to **each** linked channel, then use `/multireact CHANNEL_ID` or that channel's inline **Enable Multi React** button. The main bot verifies its own, the original user's, and every child bot's admin status before enabling. Free reaction remains available when Premium expires; extra reactions stop until Premium is renewed.

A channel with **protected content / forwarding disabled** cannot use this forward-based setup as written; forwarding must be possible for the setup post. Standard reaction emojis must be allowed in the channel. Edited posts are not processed. Already-published historical posts are not bulk-processed; only the forwarded setup post is tried once as a test.

## ⭐ Paid Multi-Bot Premium: terms and payments

- **100 Telegram Stars buys 30 days, one-time, not recurring.** `/premium` → **Buy** → read `/terms` → press **I agree** → Telegram's own Stars invoice. Each additional successful payment extends the paid time by 30 days. This is a **bot-service tier**, not Telegram Premium for the user's Telegram account and not a guarantee of custom emoji privileges. Telegram requires Stars (`XTR`) for digital services sold inside Telegram [1](https://core.telegram.org/bots/payments-stars).
- Paid checkout is **disabled until exactly five distinct owner-managed child bots initialize successfully**. Payment `pre_checkout_query` verifies the user, terms acceptance, invoice payload, price, freshness and that **all five child tokens still respond** (short timeout); **only a matching `successful_payment` activates access**. Duplicate charge IDs cannot double-grant. Paid access/refund information persists in SQLite. If an approved invoice is charged twice concurrently, each distinct successful charge gives 30 days instead of silently discarding a purchase.
- Owner may activate without payment with `/grantpremium USER_ID [days]` (30 days default); `/revokepremium USER_ID` cancels only manual access, **not paid time**. `/refundstars TELEGRAM_CHARGE_ID` uses Telegram's Stars refund API; for an **untracked** charge, owner first checks the buyer's Telegram receipt then uses `/refundstars USER_ID TELEGRAM_CHARGE_ID`. A refunded payment stops contributing to paid time. Customers can reach the owner via `/support message` or `/paysupport message`; the owner replies with `/reply USER_ID message`. Owner must `/start` the bot and monitor support in a timely fashion. Buyers should never send bot tokens, card details, or passwords in support messages.
- A user must still be an **admin of their own linked channel**; the main and **all five** child bots must also be admins. Even when enabled, Telegram/network/channel reaction restrictions may prevent one or more reactions. The bot **does not react to historical posts in bulk**, and extra bot reactions are **not organic members**. If the child-bot service becomes unavailable, owner should restore it or handle support/refunds promptly. Display the terms accurately to buyers. This is a starter implementation: review taxes, platform terms and your jurisdiction before launching a paid service.

## ✨ Premium emoji and flying celebration

- Once a non-owner passes **all** force-sub membership checks, the bot sends **one flying-emoji GIF embedded inside `bot.py`** (no separate asset file) **inside the chat message**, and tries to add Telegram's native `message_effect_id` to the **same message**. That native effect may animate confetti over the private chat [1](https://core.telegram.org/bots/api). The example 🎉 ID in the `.env` template above is community-reported [1](https://stackoverflow.com/questions/78600012/message-effect-id-in-telegram-bot-api), **not guaranteed** to remain valid. If Telegram rejects it, the GIF still sends. Telegram clients/settings may not show native effects; the bot cannot force an animation across the user's whole phone screen.
- A first-time verified user sees this once; repeatedly clicking Verify does not flood them. Losing membership and verifying again, or adding a new required channel, makes them eligible for a fresh celebration. The GIF is bundled in the Python file; Pillow is **not** a runtime dependency.
- In the celebration's text/caption the bot **tries your `tg-emoji` IDs** (`🤩` and `✈️`) with `parse_mode="HTML"`. If Telegram rejects custom emoji for this bot, it automatically retries with ordinary emoji. Custom emoji entities require an eligible bot (for example, a qualifying Premium bot owner for direct private messages, or additional purchased bot usernames) [1](https://core.telegram.org/bots/api).
- Owner can test immediately with `/effecttest`. To select another Telegram-native effect, send a message **with that effect** to your bot, then reply **to that message** with `/seteffect` (or use `/seteffect EFFECT_ID`). `/seteffect off` selects GIF-only. Alternatively set `CELEBRATION_EFFECT_ID=` blank in `.env` for GIF-only, then restart. `/seteffect` changes persist in SQLite and override `.env`. To reset an effect that fails, choose another ID and run `/effecttest` again.

**GIF attribution:** The fallback is a new animated composition made from modified [1](https://github.com/jdecked/twemoji) emoji graphics (party popper, confetti, sparkles, heart, star, and party face). Twemoji graphics copyright Twitter, Inc. and contributors; graphics licensed under CC BY 4.0. Modifications: resized, layered and animated. Attribution is also printed inside the GIF.

## Commands

| Who | Command | What it does |
| --- | --- | --- |
| Anyone | `/start` | Check required channels; open dashboard or join buttons |
| Anyone | `/help` | Explain commands and limits |
| Anyone | `/whoami` | Show your numeric user ID |
| Verified user | `/setchannel` | Start admin-verified channel linking; forward a channel post next |
| Verified user | `/mychannels` | Show linked channels and inline controls |
| Verified user | `/setreaction 🔥` | Choose or change emoji (prompts for a channel if you own several) |
| Verified user | `/setreaction -1001234567890 ❤️` | Change a specific linked channel's reaction |
| Verified user | `/pause [channel_id]` | Stop reacting to new posts in a channel |
| Verified user | `/resume [channel_id]` | Resume, after admin check |
| Verified user | `/removechannel [channel_id]` | Unlink with confirmation |
| Verified user | `/premium` | Premium status and 100-Star invoice path |
| Verified user | `/childbots` | Five child-bot usernames and setup instructions |
| Verified user | `/multireact [channel_id]` | Turn on 5 child bots after Premium/admin checks |
| Verified user | `/multireact off [channel_id]` | Turn off extras without affecting the free reaction |
| Anyone | `/terms`, `/support your question` | Purchase terms and support/refund request |
| Bot owner only | `/addforce @publicchannel` | Require membership of a public channel (bot must be admin) |
| Bot owner only | `/addforce -1001234567890 https://t.me/+invite` | Require membership of a private channel |
| Bot owner only | `/forces` | See required channels and inline removal controls |
| Bot owner only | `/delforce @publicchannel` | Remove a required channel (or pass its numeric ID) |
| Bot owner only | `/seteffect EFFECT_ID` or reply `/seteffect` | Change the private-chat celebration effect |
| Bot owner only | `/seteffect off` | Use animated GIF only |
| Bot owner only | `/effecttest` | Preview the effect/custom emojis or GIF in the owner's chat |
| Bot owner only | `/grantpremium USER_ID [days]` | Grant free manual access (default 30 days) |
| Bot owner only | `/revokepremium USER_ID` | Remove manual access; paid access is unchanged |
| Bot owner only | `/refundstars CHARGE_ID` (or `/refundstars USER_ID CHARGE_ID`) | Refund a tracked payment (or verified untracked charge) |
| Bot owner only | `/reply USER_ID message` | Reply to a support request |
| Bot owner only | `/stats` | Counts, child-bot readiness and owner panel |

If there's only one linked channel, `/pause`, `/resume`, `/removechannel`, and `/setreaction 🔥` work without a channel ID. For several channels, use their inline buttons or specify the numeric ID. Someone who is no longer a channel admin cannot change its reaction or resume it. A previously linked channel cannot be claimed by a different admin while its linked admin still has admin access (except by the bot owner).

## What are `tg-emoji` tags?

The example in your message:

```html
<tg-emoji emoji-id="5332656211034652890">📞</tg-emoji>
```

is **Telegram HTML message formatting** for a **custom emoji**. `emoji-id` identifies the custom artwork; `📞` is the ordinary fallback. A Python dict with keys `whatsapp`, `telegram`, `facebook`, `instagram`, `tiktok`, `google`, `twitter` is just a **named icon map**: it does not connect those social networks to your bot. `twitter: "🐦"` is simply a regular emoji.

- With eligible bots, this tag can be sent in **message text** with `parse_mode="HTML"`. Telegram places eligibility conditions on custom emoji entities (for example, a bot with an additional purchased username or qualifying Premium owner, depending on where the bot sends the message) [1](https://core.telegram.org/bots/api).
- **Inline keyboard button text is plain text**, not HTML; putting `<tg-emoji ...>` directly into button `text` will show the literal markup. This bot therefore uses normal emoji (`🔗`, `✅`, etc.) in inline buttons.
- A custom emoji **display ID** is not a request to add lots of reaction counts. `/setreaction` accepts **standard Telegram reaction emoji**, not these HTML tags. Channel admins must allow the chosen emoji under channel reactions.

API references: Telegram Bot API [1](https://core.telegram.org/bots/api) (setMessageReaction, getChatMember, formatting) and python-telegram-bot documentation [2](https://docs.python-telegram-bot.org/en/stable/).

## Troubleshooting

- **Verification unavailable:** bot must be admin in *each* force-sub channel, and the saved private invite link must actually let users join. Re-run `/addforce` to update a link.
- **Channel link fails:** check both admins, forward a real channel post (not a copied post, message from a group, or a post with protected forwarding), and make sure Telegram lets the bot access the channel.
- **Reaction fails:** enable reactions in the target channel and allow the chosen standard emoji; check the process logs for the failure type (raw API errors are not logged to protect tokens). The bot changes its own **single** reaction; it cannot add a second independent reaction as itself.
- **No reaction on posts:** keep `python bot.py` running; grant the main bot admin access; check `/mychannels` status; ensure no other poller/webhook consumes main-bot updates. The project requests `channel_post` updates.
- **Multi React OFF / fewer than 5 bots:** put **five distinct owner-controlled** child bot tokens in `CHILD_BOT_TOKENS` in `.env` and restart; read startup logs for slot numbers **without logging tokens**. Add **every** child bot as admin to the user's channel; then try `/multireact CHANNEL_ID`. Premium payment is disabled unless all five are online.
- **No Stars invoice:** owner must configure all five bots; user must join required channels and agree to purchase terms. The bot requests `pre_checkout_query` updates. Payments are live: Telegram charges after its checkout flow and buyer confirmation; the bot grants access only after `successful_payment` arrives.
- **Security:** rotate any exposed primary token via @BotFather before running. `.env` and SQLite are git-ignored, but protect/back up both. Never ask users to send their bot tokens, never bundle secrets in the ZIP, and do not present the extra bots as genuine human engagement.

## Local tests (optional source workspace)

The **compact ZIP omits test files** to keep the release to the minimum number of files. The source workspace retains one combined offline test script, `tests/test_bot.py`. To run it from the source workspace:

```bash
python -m unittest discover -s tests -v
```

All 32 offline tests passed before packaging. Tests mock Telegram network calls; live setup still requires a new private bot token, real channels, and Telegram admin permissions.
