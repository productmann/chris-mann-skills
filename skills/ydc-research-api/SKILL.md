---
name: ydc-research-api
description: >
  Research any topic using the You.com Research API — returns deep, multi-step
  web research with cited sources. Use this skill when the user explicitly says
  "ydc-research-api" or invokes it by name. Also use for thorough research
  requests requiring real-time sourced results — such as "research X in depth",
  "find comprehensive info on Y", or "give me a detailed report on Z".
  Prefer this skill over training data when the user wants recent, sourced, or
  comprehensive information.
allowed-tools: Bash
---

## First-Time Setup

Check whether a You.com API key is configured:

```bash
source ~/.claude/.env 2>/dev/null; echo $YDC_API_KEY
```

If the output is empty, the key is not yet set. Use the `AskUserQuestion` tool to ask:

> **You.com API Key Required**
>
> This skill needs a You.com API key to run. You can get one free at [you.com/api](https://you.com/api).
>
> Once you have it, paste your key here and I'll save it for you automatically.

Wait for the user to paste their key. Then save it:

```bash
mkdir -p ~/.claude
touch ~/.claude/.env
# Remove any existing YDC_API_KEY line and append the new one
grep -v "^YDC_API_KEY=" ~/.claude/.env > /tmp/.env_tmp && mv /tmp/.env_tmp ~/.claude/.env
echo "YDC_API_KEY=PASTED_KEY" >> ~/.claude/.env
source ~/.claude/.env
echo "Key saved: $YDC_API_KEY"
```

Replace `PASTED_KEY` with what the user provided. Confirm the key was saved, then continue.

---

## Instructions

**Step 1 — Confirm the query**

The user's research question is:

> $ARGUMENTS

If `$ARGUMENTS` is empty, ask the user what they'd like to research before continuing.

---

**Step 2 — Ask for research depth**

Always ask the user to choose a depth level before running — regardless of how the query was phrased. Use the `AskUserQuestion` tool with a single-select question and these four options:

- **Lite** — Fast, best for simple or factual questions
- **Standard** — Balanced speed and depth *(recommended)*
- **Deep** — Thorough, best for complex or multi-faceted topics
- **Exhaustive** — Most comprehensive and slowest; use for detailed analysis

Wait for their reply. If the response is unclear or doesn't match any option, ask once more before proceeding.

---

**Step 3 — Run the API call**

Map the chosen depth to its API value:

| Selection  | API value    |
|------------|--------------|
| Lite       | `lite`       |
| Standard   | `standard`   |
| Deep       | `deep`       |
| Exhaustive | `exhaustive` |

Run the request using `jq` to build the JSON body — this safely handles quotes, apostrophes, and any special characters in the query:

```bash
curl -s -X POST "https://api.you.com/v1/research" \
  -H "X-API-Key: $YDC_API_KEY" \
  -H "Content-Type: application/json" \
  -d "$(jq -n --arg q "RESEARCH_QUESTION" --arg effort "CHOSEN_LEVEL" '{input: $q, research_effort: $effort}')"
```

Replace `RESEARCH_QUESTION` with the user's query and `CHOSEN_LEVEL` with the selected API value.

---

**Step 4 — Present the results**

- Display `output.content` as markdown — it includes inline citations like [1], [2]
- Follow with a **Sources** section listing each entry from `output.sources` as: `[title](url)`

---

**Step 5 — Handle errors**

If the API returns an error, explain it clearly:

| Code | Issue                   | What to tell the user                                              |
|------|-------------------------|--------------------------------------------------------------------|
| 401  | Invalid or missing key  | Ask the user to double-check their key at you.com/api and paste it again |
| 403  | Key lacks access        | Check API plan and permissions at you.com/api                      |
| 422  | Malformed request       | The query may contain unsupported characters — try rephrasing      |
| 500  | You.com server error    | Wait a moment and try again                                        |
