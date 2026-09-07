# phishing-agentic-triage-guardrails

An n8n phishing triage agent, the email that hijacked it, and the guardrails that stopped part of it.

Companion repository for **Underground Labs #01 — How an AI phishing triage agent got phished**.
[Read the post](ADD_MEDIUM_LINK)

---

## What this is

A phishing triage agent built the way most people would build one on a first pass: an agent node, a model, two lookup tools, and one tool that sends the reply. Then an email written to talk to the agent instead of the user.

The V1 agent classified a phishing message as benign and sent its reply to an address the attacker chose. V2 restructures the workflow so the second half can't happen, and a guardrail classifier attempts the first half.

Both workflows are here so you can import them side by side.

---

## Contents

```
workflows/
  v1-naive-triage.json          the agent as first built
  v2-guardrailed-triage.json    tools removed, classifier added
  mock-mail.json                webhook target standing in for outbound mail
payloads/
  01-benign.md                  normal email, control case
  02-classic-phish.md           obvious phishing, control case
  03-injection.md               the one that worked
traces/
  v1-injection.json             execution where the agent was hijacked
  v2-injection.json             same email through V2
```

---

## Requirements

| | |
|---|---|
| n8n | 2.37.10 — workflow schema changes between releases, imports may break on other versions |
| Model | qwen3:8b |
| urlscan.io | no key needed, uses the public search endpoint |
| ip-api.com | no key needed |

Credentials are stripped from the exported workflows. You'll need to create your own model credential and select it on the chat model nodes after importing.

---

## Running it

1. Import `mock-mail.json` and **activate it**. Nothing else works until this webhook is live at `/webhook/mock-mail`.
2. Import `v1-naive-triage.json`.
3. Attach your model credential to the chat model node.
4. Open the Set node after the trigger and paste one of the payloads into `body`.
5. Run the workflow from the canvas.

The tools only execute when the agent calls them, so running individual nodes won't do anything useful. Check the Logs panel to see what the agent called and with which arguments.

For V2, repeat with `v2-guardrailed-triage.json`. Two chat model nodes need credentials there — one for the agent, one for the guardrail.

---

## About the payloads

`03-injection.md` contains a working indirect prompt injection. URLs are defanged in the markdown files (`hxxp://`, `[.]`) so they don't trip mail scanners or get clicked by accident. **The version inside the workflow JSON is not defanged**, because the agent needs a real domain to look up. If you copy a payload from `payloads/` into a workflow, undefang it first.

These are research artifacts. They're plain text, they don't execute anything, and they exist to demonstrate a documented weakness in agent architectures.

---

## What V2 fixes and what it doesn't

**Fixed.** The reply recipient comes from the trigger rather than the model. The agent has no tool that sends mail, so there's nothing in the email body for that instruction to reach.

**Not fixed.** The verdict is still a judgement the model makes about attacker-written text. The guardrail classifier raises the cost of getting past it. It does not close it, and in the run captured in `traces/v2-injection.json` it did not catch this payload.

---

## Notes for anyone rebuilding this

A few things that cost me time:

- Tool parameters filled by the model need the field set to **Expression** mode. The `=` prefix is added by n8n — typing it manually sends it as a literal.
- Webhook **Path** takes the path segment only. `mock-mail`, not `/webhook/mock-mail`.
- `/webhook-test/` requires the receiving workflow to be armed for a single call. `/webhook/` needs it activated. Mixing them up produces a 404 that looks like the webhook doesn't exist.
- `[undefined]` in an expression preview is expected. Model-filled parameters only resolve when the agent actually calls the tool.
- Writing a guardrail prompt that quotes well-known attack phrases can get the prompt itself blocked by upstream content filters. Describe the pattern instead.

---

## License

MIT for the workflows and code. The blog post text is not covered.