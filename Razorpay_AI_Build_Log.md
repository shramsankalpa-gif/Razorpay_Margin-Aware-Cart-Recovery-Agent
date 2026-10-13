# AI Build Log: Margin-Aware Cart Recovery

Razorpay × ISB AI PM Build Challenge · Track 2: Recover and Grow with AI
Builder: Anuraag Praharaj (ISB PGP Co'27) · Prototype: https://razorpaymargin-aware-cart-recovery.vercel.app/ · Deck: https://razorpaymargin-aware-cart-recovery.vercel.app/deck/

Concept only, not a Razorpay product. All merchant data is synthetic. No API keys, credentials, private or proprietary data are included in this repo or this log.

---

## 1. AI tools and models used

| Tool | Used for |
|---|---|
| **Claude (Anthropic)**, chat with a project knowledge base (the challenge brief, two job descriptions, course notes) | Research, critique of ideas, drafting the product note and deck copy |
| **Claude agentic workspace** (code execution, file tools, headless browser) | Wrote and tested the prototype, deck and note; rendered every screen and slide to check layout |
| **Claude web search / fetch** | Read Razorpay's public docs and blog (Magic Checkout offers and abandoned-cart webhook, Agent Studio launch and guardrails) and press coverage |
| **Claude live-AI call inside the prototype** (only when opened in claude.ai) | Reads an uploaded margin file (Excel/PDF) into the setup form; answers merchant questions on the Recommendations screen. Elsewhere the page falls back to computed answers |

Open-source libraries in the prototype: Chart.js 4.4.1 (charts), SheetJS 0.18.5 (Excel parsing), pdf.js 3.11.174 (PDF parsing), all loaded from a public CDN. Testing: Playwright and jsdom. Fonts and colours from Razorpay's public Blade design-system package. No other AI model or external API is called.

## 2. Key prompts, workflows and agent setup

**Workflow.** One long conversation with Claude, in order: research → idea stress-test → V1 to V2 of the idea → synthetic model → prototype → deck → note. At every stage I gave a direction in plain language, Claude produced a draft, and I corrected it. I asked Claude to label each claim as factual, somewhat factual or guess.

**Representative prompts (paraphrased):**
- "Do holdouts receive a reminder or not? How do Magic Checkout and the cart agent work?" Used to establish what Razorpay actually offers before designing anything.
- "Who decides which coupon is offered, the agent or the merchant?" Led to the framing that Magic Checkout holds the merchant's coupons and the cart agent picks among them.
- "Drop to the top-5 margin categories, merchants share only margins, randomise discounts in 0.5% steps with a 10% zero-discount group." The V2 scope.
- "Is AI really present in this solution? Where exactly, and why does it beat a fixed rule?" Forced the explicit AI-versus-rule section.
- "Build a prototype with an Agent Studio-style interface, three merchants, PDF/Excel upload, a clickable feature list."
- "Fact-check the deck: is 'one coupon for every cart' accurate? Is 'who gets paid: everyone' accurate?" See section 3.

**Agent design (what the product does, built with Claude's help):**
- Trigger: Magic Checkout's abandoned-cart event plus the payment-attempt result, only if the cart holds one of the merchant's top-5 margin categories.
- Input: 10 bucket-level signals (first-time/repeat, payment method, payment status, time slot, day type, season, region tier, cart size vs store AOV, coupon behaviour, UTM source) plus categories in the cart.
- Decision: for each discount from 0 to the cap in 0.5% steps, predict purchase probability c(d) and choose the d that maximises c(d)·(m − d), within the merchant's cap and margin floor.
- Experiment: 10% reminder-only control, random test offers while learning, then optimised offers. In the prototype the explore → learn → optimise phases are simulated. A contextual bandit is the intended production design.
- Guardrails: merchant owns cap and floor, review before going live, per-signal switches, pause, audit log.

## 3. One AI output I rejected, corrected and improved, and why

**The deck claimed that today's tools give "one coupon for every cart".** Claude wrote this when drafting slide 5. I doubted it and asked for a fact-check. Claude re-read Razorpay's public documentation: Magic Checkout coupons are merchant-created (fixed, percentage, buy-X-get-Y, tiered by cart value), and the Agent Studio cart agent picks among the merchant's preset offers up to a merchant-set maximum. So the claim was wrong. It overstated the gap and would have been easy for a Razorpay reviewer to spot.

**Fix:** the deck now says "preset menu: flat or tiered by cart value". The real gap is stated instead: public material describes no control group, margin floor or incremental-profit report. The same wrong claim was also on slide 1 and was fixed there. In the same pass I rejected "everyone gets paid" for the flat-vs-tiered row. It is now "every shopper who redeems it, including those who'd have returned on a reminder alone".

**Other corrections:**
- Claude proposed coarse test levels (0 / 5 / 10%). I chose fine 0.5% steps, and we fit a response curve so it stays statistically workable.
- Claude proposed "drop-off stage" and "past coupon use" as signals. I cut both: the first isn't reliably available, the second needs customer-level data.
- Claude corrected me too: hashing is not encryption and does not solve data sharing (contracts and opaque IDs do), and 31 category combinations cannot each be tested, so one joint model is used.

## 4. What AI did versus what I decided or built

**AI did:** read Razorpay's public docs and summarised them; wrote the synthetic data generator, response model and prototype code; computed the flat / tiered / agent comparisons; laid out the deck and note in Razorpay's design tokens; ran automated checks.

**I decided:**
- Track 2, abandoned carts, **profit rather than conversion** as the objective.
- Only the **top-5 margin categories**; discount applies to those items only in mixed carts.
- **Randomised 0.5% steps** plus a **10% zero-discount control group**.
- Offers shown **in rupees**.
- The **signal set**, and what to exclude: bucket-level data only, **no income (customer-level personal data), no device**.
- The merchant **keeps the cap and floor** (business explainability); new merchants start from Razorpay's existing pooled data.
- Putting the merchant's **next best action upfront** in the prototype, plus a plain-language "How it works".
- What to leave out, and the claims I was willing to put my name to.

**I did myself:** recorded the demo video, deployed to GitHub and Vercel, and submitted.

## 5. Known limitations, risks and verification steps

**Limitations (stated plainly):**
- **All data is synthetic.** The three merchants, about 14,000 carts and the response curves come from a simulation I calibrated, not from real shoppers. The headline numbers (for example +₹52.8k per 1,000 carts for the agent vs +₹37.5k for flat 10% on the skincare merchant) show the mechanism, not an expected real-world gain.
- **FuelUp's flat 10% loses money (−₹15.1k) in the simulation.** That depends on how the synthetic margins and response were set. It illustrates the failure mode but is not evidence about any real store.
- **The straight-through conversion rate α = 55% is an assumption**, used for the illustrative maths.
- **The bandit is simulated.** The explore → learn → optimise phases are scripted in the prototype. A real contextual bandit is the intended design and is not built.
- **Live AI works only inside claude.ai.** On Vercel, the upload-reading and Q&A features use computed answers.
- **Razorpay internals are unknown to me.** I used only public documentation. I could not confirm Magic Checkout or the cart agent's internals, and public channel descriptions (voice, WhatsApp, email) are inconsistent.
- The prototype was tested with automated checks (jsdom, headless Chromium screenshots) and a manual click-through of the deployed site by me. It was not tested across multiple browsers.

**Risks:** discount gaming (mitigated by a per-contact offer cap and random ₹0 offers); drift (stop-loss if a group falls below control); thin data (fall back to the merchant's own rule); fairness (income and device excluded; customer-level signals switchable); DPDP consent before any message.

**Verification steps taken:**
1. Every claim about Razorpay was checked against its public docs and blog; unverifiable claims were labelled as assumptions or removed.
2. The model code was run in an automated harness with no errors. Period selector, merchant switch, upload and diff, signal toggles and recommendations were each exercised.
3. Profit maths was recomputed in code, not by hand. The formula and the "discount pays only if (c(d) − c(0))·(m − d) > c(0)·d" condition are used throughout.
4. Every deck slide and the note were rendered and checked for text overflow.
5. I fact-checked the deck line by line (section 3) and deployed and clicked through the live site before submission.
