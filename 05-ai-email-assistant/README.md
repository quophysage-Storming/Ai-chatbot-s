# AI Email Assistant: sort leads and draft replies. Free n8n template (Bot Playbook, Episode 5)

Every new email gets read by an AI and sorted as a **Lead**, **Support** or **Spam**. Leads get a Lead label and a reply draft, support emails get a reply draft, and spam is marked as read. Nothing is ever sent: the AI only writes drafts, so you read them and hit send yourself. This is the exact workflow built in the video.

| File | What it is |
|---|---|
| [`ai-email-assistant.json`](ai-email-assistant.json) | Gmail Trigger → Basic LLM Chain (Gemini + Structured Output Parser) → Switch → Gmail: add label, create a draft, mark as read. |

| Node | What it does |
|---|---|
| Gmail Trigger | Checks every minute. Simplify **off** (the AI needs the whole email), Read Status: unread only, Search: `-from:me` |
| Basic LLM Chain | Reads the sender, subject and the first 3,000 characters of the body, with the rules in its System message. **Retry On Fail** is on (5 tries, 5 seconds apart), because AI services are sometimes busy for a moment |
| Google Gemini Chat Model | The AI. Swap it for OpenAI or Anthropic if you like: the chain works the same |
| Structured Output Parser | Forces the answer into a form: `category` (only Lead, Support or Spam), `reason`, `reply` |
| Switch | Three exits on `{{ $json.output.category }}`: Lead, Support, Spam |
| Add label to message | Lead only: adds your Lead label |
| Create a draft | Lead and Support: the AI's reply as a draft, inside the original conversation (Thread ID) |
| Mark a message as read | Spam only |

## Set it up in 15 minutes

1. **Import:** in n8n, open a new workflow, then **⋯** → **Import** → **From URL**:
   ```
   https://raw.githubusercontent.com/shafiqa123personal-do/bot-playbook-templates/main/05-ai-email-assistant/ai-email-assistant.json
   ```
2. **Gmail:** open each of the four Gmail nodes and pick your Gmail credential. On n8n Cloud, create it with **Sign in with Google**. If you host n8n yourself, follow n8n's Google OAuth guide first: https://docs.n8n.io/integrations/builtin/credentials/google/oauth-single-service/
3. **Your Lead label:** make a label called Lead in Gmail and give it a colour, then pick it in **Add label to message**.
4. **The AI:** in **Google Gemini Chat Model**, add your Gemini API key. Or delete it and connect an OpenAI or Anthropic Chat Model instead.
5. **Your details:** in **Basic LLM Chain**'s System message, replace *Sam Ortiz, a freelance automation consultant* with your name and what you do, and `https://example.com/book-a-call` with your own booking link.
6. **Test** with a test inbox first: send it an email from another account, click **Fetch Test Event** on the Gmail Trigger, then **Execute workflow**.
7. **Publish.**

## Before you use it on a real inbox

- **It never sends.** Keep it that way: drafts mean the worst case is a bad draft you delete. Anyone can email you, and some emails will try to give the AI orders. The System message tells it to treat emails as information, not instructions, but drafts are the real safety net.
- **Privacy:** every email is sent to the AI company to be read. On Gemini's free plan, Google may use what you send to improve its products. For client email, use a paid plan, and check the terms of whichever AI you pick.
- **Long newsletters** are cut to 3,000 characters before the AI reads them, to keep each run small.
- **Busy inboxes:** Gemini's free tier allows only a few requests a minute (5 a minute on Flash when we tested, November 2026). A very busy inbox needs a paid plan.
- **Wrong sorting?** Look at the `reason` the AI gave in that execution, then fix the matching rule in the System message.

Made with n8n 2.44. Free to use, change, and use for your clients ([MIT License](../LICENSE)).
**Video:** [youtube.com/@BotPlaybook](https://www.youtube.com/@BotPlaybook) · **Want it built for you?** [fiverr.com/systemsyncai](https://www.fiverr.com/systemsyncai)

Not affiliated with Google, Gmail or n8n.
