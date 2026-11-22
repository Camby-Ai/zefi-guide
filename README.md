# Zefi Setup Guide

Zefi is a travel assistant. Planning a trip is a dozen tabs, three group chats and one person who ends up doing all of it. Zefi takes it: you give her a budget range and a vibe, she researches the flights and the stays, builds the whole itinerary around the budget, checks it against the second traveler's rules before anything is booked, and keeps one page of every event worth going to this year. This guide sets her up on Hermes Agent, a free and open source agent from Nous Research that runs on your own machine.

The software is free. The model behind it is not always free, and the real work is writing down how you like to travel once, in one place.

**Checked on 2026-11-21.** Commands and limits change. Sources are at the bottom.

**How this guide was made:** every command below was copied from the Hermes Agent documentation on that date. We have not test-run them on a fresh machine. If a step does not behave the way it says, see "If a step breaks" near the end: your AI can patch it for you in a minute.

**Zefi plans. You book.** In this guide she never buys a ticket, holds a room or enters a card. Prices and schedules she finds are a snapshot from the moment she looked: check them on the airline's or hotel's own page before you pay.

## What Zefi does

- Keeps one page of how you travel: home airport, seats, hotel style, what you never do.
- Takes a budget range and a vibe and builds the whole trip around it: flights, stay, dinners, plans, with the running total against your range.
- Gives you her top three options for each flight and stay, with the link and the date she checked.
- Keeps a second page for the person you travel with, and checks every plan against both pages before she calls it ready.
- Keeps one page of the year: every conference, festival and gathering worth being at, with dates and what has to be decided by when.
- Keeps a guest list for a group trip: who is invited, who said yes, what each person still owes an answer on.
- Tells you each Monday what needs a decision this week.
- Does not book, pay, hold, cancel or message anyone. You do.

## What you need before you start

| You need | Why | Cost | Checked |
|---|---|---|---|
| A computer that stays on: macOS, Linux, Windows or WSL2 | The Monday note only runs while Hermes' gateway is running | Yours already | 2026-11-21 |
| Git and curl | The installer uses them | Free | 2026-11-21 |
| Hermes Agent | Runs Zefi | Free, MIT licence | 2026-11-21 |
| A model with at least 64,000 tokens of context | Hermes rejects smaller models at startup | Depends on the provider you pick. Not measured here | 2026-11-21 |
| Web search | She has to look things up | Hermes' built-in `web_search` and `web_extract` tools. The docs say a fresh install works with no web key at all, on free tiers with limits | 2026-11-21 |
| Telegram (optional) | To get the Monday note on your phone | Free | 2026-11-21 |

On a brand new install the setup wizard may offer a "Free, start now" choice that needs no account. The Hermes docs say it is shown only when it is available, so do not plan around it.

## The honest timeline

| When | What | Your time |
|---|---|---|
| Day 1 | Install Hermes and get one normal chat working | The Hermes docs say under 5 minutes. Allow 30 |
| Day 1 | Create Zefi and give her a desk folder | 20 minutes |
| Day 1 | Write how you travel, and how your travel partner does | 30 to 45 minutes. This is the part that decides everything |
| Day 2 | Turn on the Monday note | 20 minutes |
| Week 1 | Plan one real trip with her, start to finish | An hour, most of it reading her options |

The slow part is not the install. It is saying out loud what you actually want from a trip. "Somewhere warm, not too expensive" gets you a generic answer. A budget range, a home airport and three things you never do gets you a trip.

Three limits to know before you start:

- She reads public web pages. She cannot see member-only fares, your loyalty accounts or your seat map, and a price can change between her search and your click.
- She does not read your calendar or your inbox in this guide. Tell her your dates, or see the optional note under step 4.
- She cannot invite anyone. She drafts the invitation and keeps the list. You send it.

## Setup, in this order

### 1. Install Hermes and get one chat working

Do not add anything else until a plain conversation works. The Hermes quickstart is blunt about this, and it is right.

On macOS and Windows the Hermes docs recommend the Hermes Desktop installer from https://hermes-agent.nousresearch.com/. For a command line only install on macOS, Linux or WSL2:

```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

Reload your shell, pick a model, and start a chat:

```bash
source ~/.zshrc
hermes setup
hermes
```

You should see: a welcome banner naming your model, and a reply to a simple question. If replies are empty or broken, run `hermes model` again and confirm the provider, the model and the login. If anything else looks off, run `hermes doctor`.

### 2. Create Zefi

Every agent in Hermes gets its own profile: its own settings, memory and identity. Never point two agents at the same profile.

```bash
hermes profile create zefi --description "Travel assistant. Builds a whole trip around a budget range, gives the top three flight and stay options with links, checks every plan against both travelers' rules, keeps the year's events and a group's guest list. Never books, pays, holds or messages anyone. The owner does."
```

Creating the profile also creates a `zefi` command. Check it:

```bash
zefi chat
```

You should see: the same kind of chat as step 1, now running as the `zefi` profile.

### 3. Give Zefi her identity

In Hermes, who an agent is lives in a file called `SOUL.md` inside that agent's home. For a profile named `zefi` that is:

```text
~/.hermes/profiles/zefi/SOUL.md
```

Hermes creates a starter file there. Replace its contents with the text in "Zefi's instructions" below. Keep it about who she is and how she talks. How you travel goes in different files, in the next step.

You should see: after starting a new `zefi chat`, she introduces herself as your travel assistant when asked who she is.

### 4. Build her desk: the rules, how you travel, the year

This is the step everybody rushes, and it is the one that makes Zefi useful.

Make a folder for her to work from:

```bash
mkdir -p ~/zefi-desk/trips ~/zefi-desk/groups
```

Put three files in it. Hermes loads `AGENTS.md` from the folder you start a chat in, and Zefi reads the others as she works.

`~/zefi-desk/AGENTS.md`, the rules:

```text
# How Zefi works this desk

- travelers.md says how each of us travels. Read it before every plan. Never guess a preference that is not written: ask me once and write the answer down.
- Every trip gets one file in trips/, named for the place and the month. It starts with the brief: dates, who is going, the budget range, the vibe.
- Build the whole trip around the budget range: flights, stay, dinners, plans. Keep a running total and show it against the range at the top of the file.
- For each flight and each stay give me your top three, not thirty. For each one: what it is, the price you saw, the link, and the date and time you checked. A price with no link and no date does not go in the file.
- Before you call a plan ready, check it against every traveler in travelers.md. List anything that breaks one of their rules. A plan is ready only when it works for everyone going.
- year.md lists every event worth being at this year. For each one keep the dates, the place, what has to be decided and by when. Add an event only with its own page as the source.
- For a group trip keep one file in groups/: who is invited, who said yes, who has not answered, and what each person still owes. Draft the invitation for me. Never send it.
- Mark everything that needs my decision with DECIDE at the start of the line.
- You plan. You never book, pay, hold, cancel or message anyone, and you never ask me for a card number, a passport number or a password. If a site asks for one, stop and tell me.
- Prices and schedules change. Say so every time you show me one.
```

`~/zefi-desk/travelers.md`, how each of you travels:

```text
# Travelers

## {your name}
- Home airport: {airport}
- Flights: {nonstop only, no red eyes, aisle seat, cabin}
- Stays: {hotel or rental, neighborhood over price, what matters}
- Food: {anything to plan around}
- Never: {the things that ruin a trip for you}
- Usual budget range for a weekend: {low to high}
- Usual budget range for a week: {low to high}

## {travel partner's name}
- Home airport: {airport}
- Flights: {their rules}
- Stays: {their rules}
- Food: {their rules}
- Never: {their rules}
- Dates that are off limits: {work, family, standing plans}
```

`~/zefi-desk/year.md`, the year:

```text
# The year

Last checked: {date}

| Event | Dates | Place | Why go | Decide by | Status |
|---|---|---|---|---|---|
| {event} | {dates} | {city} | {who will be there, what it is for} | {date} | {thinking, going, booked, skipped} |
```

You should see: when you run `zefi chat` from inside `~/zefi-desk` and ask where you fly from or what your partner never does on a trip, she answers from `travelers.md` or says it is not written.

Optional: Hermes ships a Google Workspace skill that can read a calendar. This guide does not set it up. If you want her to see your dates herself, ask her in chat to walk you through that skill's setup, and read what access it asks for before you agree.

### 5. Fill in how you travel

Run `zefi chat` from inside `~/zefi-desk` and send her this:

```text
Help me fill in travelers.md. Ask me one question at a time about how I travel: home airport, flight rules, where I like to stay, food, the things I never do, and my usual budget range for a weekend and for a week. Then ask the same about the person I travel with, and which dates are off limits for them. Write my answers into travelers.md exactly as I give them. If I do not know something, write "unknown" and move on.
```

You should see: a page for each traveler in your own words, with nothing guessed.

### 6. Schedule the Monday note

Hermes has a built-in scheduler. Give the job Zefi's folder so she reads your trips when she runs. The folder must be written as a full path.

```bash
hermes -p zefi cron create "every monday 8am" \
  "Monday travel note. Read year.md and every file in trips/ and groups/. List only what needs me this week: any DECIDE line, any event whose decide-by date is within 14 days, any trip inside 30 days with a flight or stay not marked booked, and anyone in a group who has not answered. One line each. Search nothing and change no file. If nothing needs me, reply with the single line: Nothing this week. Keep it under 150 words." \
  --workdir /Users/you/zefi-desk \
  --name "Monday travel note"
```

Schedules run from a gateway, and each profile has its own. Start Zefi's, and check that her scheduler is alive:

```bash
zefi gateway start
hermes -p zefi cron status
```

`zefi gateway install` keeps it running after a restart.

By default the note is saved on your machine. To get it on your phone, connect Telegram first with `hermes gateway setup`, then create the job with `--deliver telegram` added.

One thing to check on your machine: whether your times come out in your own timezone. Hermes reads the schedule in the profile's `timezone:` setting. Add `--timezone America/Los_Angeles` (or your zone) to pin it.

You should see: a short travel note waiting for you the next Monday morning, or the one line saying nothing needs you.

### 7. Plan your first trip, by hand

Pick one real trip. From inside `~/zefi-desk` run `zefi chat` and send her this:

```text
New trip. Where: {place, or "somewhere that fits this vibe"}. Dates: {dates or a window}. Who: {names from travelers.md}. Budget range for the whole trip: {low to high}. Vibe: {three words}. Open a file for it in trips/ starting with this brief. Search the web and give me your top three flights and top three stays, each with the price you saw, the link, and when you checked. Then draft the days: dinners and plans that fit the vibe and the budget. Check all of it against everyone in travelers.md and list anything that breaks a rule. Show the running total against my range. Book nothing.
```

You should see: a new file in `trips/` that opens with your brief, three flights and three stays with links and check times, a day by day draft, a list of anything that breaks a traveler's rule, and a total against your range.

## Zefi's instructions

Paste this into `~/.hermes/profiles/zefi/SOUL.md`. Change the parts in braces.

```text
# Who you are

You are Zefi. You are the travel assistant for {your name}.

Your job is to take a budget range and a vibe and hand back a whole trip that works for everyone going: flights, stay, dinners and plans, with the links, the total, and a short list of what still needs a decision.

# How you work

- The budget range is the frame. You build inside it and you show the total against it.
- Top three, never thirty. Each option has a price, a link and the time you checked.
- You plan for everyone going. A trip that breaks one traveler's rule is not ready, and you say which rule.
- You keep the year in view: the events worth being at, and the date each one has to be decided by.
- For a group you keep the list: invited, yes, no answer, and what each person still owes.
- You never book, pay, hold, cancel or message anyone. Your owner does.
- You never ask for a card number, a passport number or a password.
- Prices and schedules change, and you say so every time you show one.

# How you talk

- Breezy, sunny and worldly. Relaxed, like the trip has already started.
- Short. The pick first, then why in one line.
- Mark anything that needs a decision so it cannot be missed.
- When nothing needs your owner, you say so in one line and stop.
- When you ask a question, ask exactly one.
```

This persona was written from Zefi's job description for this guide. It is not a copy of the instructions she runs on in our own team.

## First test

Run `zefi chat` from inside `~/zefi-desk` and send her this:

```text
Read travelers.md and tell me, for each traveler: their home airport, their flight rules, and the things they never do. Then tell me which lines are still "unknown" or missing. Then read year.md and list every event with a decide-by date in the next 60 days. Use only what is in the files. Search nothing and change nothing.
```

A good answer repeats each traveler's rules exactly as written, names the gaps instead of filling them, and lists only the events that are on your page.

If she states a preference that is not in the file, tighten the first rule in `AGENTS.md` and ask her where it came from.

## When it goes wrong

| What you see | Why | Fix |
|---|---|---|
| Empty or broken replies | Model or login is wrong | `hermes model`, then a plain chat before anything else |
| She does not know how you travel | The chat was not started in `~/zefi-desk` | Start `zefi chat` from that folder |
| She says she cannot search | The web tools are off for her profile, or the free tier ran out | `hermes -p zefi tools` to check the web tools and pick a provider |
| A price is wrong when you click through | It changed after she looked | Normal. Check the time on her line, and ask her to search that one again |
| She gave thirty options | The top three rule is not being read | Start the chat from `~/zefi-desk`, and tighten that rule in `AGENTS.md` |
| The plan ignores your partner's rules | Their section in `travelers.md` is thin | Fill it in with step 5, then ask her to check the trip again |
| No Monday note | Zefi's gateway is not running, or the job is missing its folder | `hermes -p zefi cron status`, `zefi gateway start`, and recreate the job with `--workdir` |
| The note arrives at the wrong hour | The schedule is running in another timezone | Recreate the job with `--timezone` set to your zone |
| Anything else | | `hermes doctor` |

## If a step breaks

Hermes changes fast, so a command here can drift from what your version expects. You do not need us to fix it. Paste this into Hermes itself, or into Claude or ChatGPT, with the error underneath:

```text
I am following a setup guide for a travel assistant agent on Hermes Agent. This step failed. Here is the step, the exact command I ran, and the full error. Check the current Hermes Agent docs at https://hermes-agent.nousresearch.com/docs, tell me what changed, and give me the corrected command.
```

`hermes doctor` is worth running first. It catches most setup problems on its own.

## What it costs to run

- Hermes Agent: free, MIT licence.
- The model: billed by whichever provider you choose in `hermes model`. Every search, every plan and every Monday note is model usage. Not measured here, so no number is given.
- Web search: the Hermes docs say a fresh install searches on free tiers with no key, with limits. A paid search provider is optional and bills separately.
- Flights and hotels are yours to pay for. This guide never touches a payment.

## Want it set up for you?

CAMBY Ai sets Zefi up and runs her for you: https://mikecamby.com

## Credit

Zefi runs on [Hermes Agent](https://github.com/NousResearch/hermes-agent), built by Nous Research and released under the MIT licence. Hermes is not ours. This guide is a walkthrough for using it as a travel assistant. Not affiliated with or endorsed by Nous Research, Telegram, Google or any airline, hotel or booking site.

## Sources

All read on 2026-11-21 from the Hermes Agent repository and its documentation.

- https://github.com/NousResearch/hermes-agent : install command, first-run commands, licence.
- https://hermes-agent.nousresearch.com/docs/getting-started/quickstart : install paths, `hermes setup`, the 64,000 token minimum, the "Free, start now" option, `hermes doctor`.
- https://hermes-agent.nousresearch.com/docs/user-guide/profiles : `hermes profile create`, `--description`, one profile per agent, the per-profile command.
- https://hermes-agent.nousresearch.com/docs/user-guide/features/personality : `SOUL.md`, where it lives and what belongs in it.
- https://hermes-agent.nousresearch.com/docs/user-guide/which-file-does-what : `SOUL.md` versus `AGENTS.md`.
- https://hermes-agent.nousresearch.com/docs/user-guide/features/web-search : `web_search` and `web_extract`, the keyless free tiers on a fresh install, choosing a provider in `hermes tools`.
- https://hermes-agent.nousresearch.com/docs/user-guide/features/cron : `cron create`, the schedule wording, `--workdir`, `--deliver`, `--name`, `--timezone`.
- https://hermes-agent.nousresearch.com/docs/user-guide/messaging : `hermes gateway setup`, `start`, `install`, `status`.
- https://hermes-agent.nousresearch.com/docs/user-guide/multi-profile-gateways : one gateway per profile, `<profile> gateway start`, `hermes -p <profile> cron status`.
