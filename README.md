
# ClearThink

A student-led initiative teaching people to spot logical fallacies, built around **Busted!**, a free web app that scans writing for fallacies using a custom fine-tuned AI model.

Founded by Sudarshan Manikantan, IBDP student and national-level debater.

---

## What's in this repo

| File | What it is |
|---|---|
| `index.html` | The ClearThink homepage. Mission, events, workshops, media coverage, testimonials, contact. |
| `about.html` | The founder's About page (bio, photo, credentials). Linked from the homepage nav. |
| `busted_fallacy_guide.html` | The Busted! app itself: a fallacy field guide, quiz, and AI-powered writing scanner. Also embedded inside `index.html` via an iframe. |

These three files reference each other by relative filename (`index.html`, `about.html`, `busted_fallacy_guide.html`), so they need to stay in the same folder together, wherever they're hosted.

---

## How it's hosted

**GitHub Pages**, serving straight from the repo root. No build step, no server, no framework, just plain HTML/CSS/JS files.

Settings that matter:
- Repo must be **Public** (Pages on private repos needs a paid GitHub plan)
- Settings → Pages → Source: **Deploy from a branch** → `main` → `/ (root)`
- **Enforce HTTPS** should stay checked (Busted's API calls need HTTPS to work in the browser)

Live URL is normally `https://<your-username>.github.io/<repo-name>/`

If you ever want a custom domain, that's a separate field in Settings → Pages, paired with a DNS record from wherever you buy the domain.

---

## The AI model behind Busted!

This is the part that's easy to forget: **the trained model does not live in this repo or on GitHub at all.**

- The model is a fine-tuned `roberta-base` classifier trained to detect 12 categories of logical fallacy (plus "No Fallacy").
- Test accuracy: 98.6%.
- It's hosted as a **Hugging Face Space**: `Sudhirulz/LogicalFallaciesDetector`
- Live URL: `https://sudhirulz-logicalfallaciesdetector.hf.space`
- It runs on Hugging Face's free **ZeroGPU** tier via Gradio.

When someone uses the Busted! scanner, the browser makes a live API call over to that Hugging Face Space. The website (GitHub Pages) and the model (Hugging Face) are two separate, independently-hosted things connected only by that API call:

```
[GitHub Pages: HTML/CSS/JS]  →  fetch()  →  [Hugging Face Space: trained model]
```

**Practical implications:**
- If the Hugging Face Space is ever deleted, or the free tier changes, or the account lapses, the scanner breaks even though the website itself is fine.
- ZeroGPU sleeps when idle. First scan after a quiet period can take 30-60 seconds while the model wakes up. This is normal.
- The Hugging Face account is a separate login/dashboard from GitHub. Two places, not one.

---

## A known limitation: the "specific explanation" feature

Busted!'s scanner does two things per sentence:
1. Classifies the fallacy (via the Hugging Face model)
2. Writes a sentence-specific explanation and rewrite suggestion (via a second call to the Anthropic API)

Step 2 currently only works inside the Claude.ai artifact environment, which proxies that API call for free without needing a real API key. **Once hosted on GitHub Pages (or anywhere outside Claude.ai), that second call will fail**, because Anthropic's public API doesn't allow direct unauthenticated browser calls.

What happens in practice: the fallacy detection itself keeps working fine. Only the specific, tailored explanation degrades, falling back to generic pre-written explanations per fallacy type instead of one written for the exact sentence.

To fix this properly would require a small backend (e.g., a serverless function) that holds a real Anthropic API key and proxies the request, so the key is never exposed in the browser. Not done yet, flagged here for later.

---

## Model training details (for reference)

- **Architecture:** `roberta-base`, fine-tuned, 12-class sequence classification
- **Classes:** Affirming the Consequent, Hasty Generalization, Appeal to Nature, No True Scotsman, Texas Sharpshooter, Ad Hominem, Straw Man, False Dilemma, Slippery Slope, Appeal to Authority, Circular Reasoning, No Fallacy
- **Dataset:** 480 training examples, 72 test examples, 46 per class, hand-built with template sentences, essay-style examples, and hard-negative pairs
- **Test accuracy:** 98.61%, macro F1 0.9860

---

## Ideas for later

- Build a small backend proxy so the specific-explanation feature works outside Claude.ai
- Connect the newsletter signup form to something real (Google Forms, Mailchimp)
- Replace the placeholder Media Coverage and testimonial cards with real quotes as they come in
- App Store / Google Play listings (currently "Coming Soon")
- More event photos and workshop write-ups for the homepage

---

*This README is for whoever needs to pick this project back up later, including future-you.*

