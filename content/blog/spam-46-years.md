+++
title = "46 years of spam: history, anatomy, and how to fight it"
date = "2025-11-03"
+++

Notes from my talk on the history of spam, how to dissect the “perfect” spam email, modern tools to fight it, and my personal workflow, to fight it. [YouTube video](https://www.youtube.com/watch?v=zYzwp-BVXS8&list=PLK3T2dt6T1fdP_0qT5w_Q_UVpvMv410xG&index=13).

# Where “spam” came from

The word started with an American company pushing sales of its spicy ham, popular with soldiers in World War II. The modern sense of *unwanted, repetitive junk* comes from a Monty Python sketch: Vikings in a café sing “spam” over and over—as annoying as the emails we get today. Here is the sketch [YouTube video](https://www.youtube.com/watch?v=anwy2MPT5RE).

# A short history of email spam

* **1978** — Gary Thuerk sent the first sales email over ARPANET. It reportedly drove about $20 million in sales and kicked off the whole practice.
* **1991** — The World Wide Web was born; spam spread with it.
* **2003** — President Bush signed the CAN-SPAM Act. It didn’t stop spam.
* **2004** — Bill Gates predicted spam would soon be a thing of the past. 
* Notable episodes: an Australian man sentenced for Nigerian spam; the death of Vardan Kushner, known for massive spamming all people speaking Russian.

So we’re at **46 years** of spam—and counting.

# The “perfect” spam email

Modern spammers often call it “cold email” and treat sending it as a “meta skill.” A classic spam email usually has:

1. **Subject line** — Built to get a click, often tuned to cultural context.
2. **Personalized compliment** — Keeps the recipient reading.
3. **Pitch** — e.g. the “Nigerian priest in need” or similar story.
4. **Call to action** — e.g. “transfer bitcoins” or send money.

Understanding this structure helps both when judging emails and when designing filters.

# Technical defenses

* **SPF** — DNS record listing which servers are allowed to send mail for a domain (visible in Gmail and similar).
* **DKIM** — Digital signature so the message can’t be trivially spoofed.

These don’t stop all spam, but they make impersonation and some abuse harder.

# Early “AI” vs spam: Paul Graham and tokenization

Paul Graham (YC co-founder) tried to reduce spam using Bayesian filtering. He computed the *probability* that an email was spam from its content. It worked well because most messages are either clearly spam or clearly legitimate; the gray area is small. The link to [Graham's blog post](https://paulgraham.com/spam.html).

# Modern tools

* **Hey** — Uses a “screener,” blocks new domains and tracking pixels, and gives you more control over what hits the inbox.
* **Superhuman** — Relies on AI and keyboard shortcuts to triage and process email quickly.

Different tradeoffs, but both aim to put you back in control of the stream.

# A more advanced personal workflow: “everything is spam until proven otherwise”

One approach is to treat every email as unwanted by default:

* **Whitelist** — Trusted contacts go through as normal.
* **Everything else** — Handed to a Python script driven by an LLM. A custom prompt scores each email from 0–10; messages scoring 5 or higher go to a “dumpster” folder.

You build a personal spam corpus and can get a summary of binned mail (e.g. via Slack). It could cost on the order of **$10/year for 10,000 emails** as for 2025.

You get automated triage, a tunable threshold, and low cost.

# Who decides what’s spam?

What counts as spam is **personal**. You should own your own filters. There’s no perfect solution—you might miss an important message or let some spam through—but the peace of mind from controlling your inbox is worth it.

Fun fact: in Hawaii, sushi with Spam is considered a delicacy.
