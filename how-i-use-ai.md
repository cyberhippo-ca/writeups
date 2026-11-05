# How I Use AI: From Prompts to Daily Automations

*Posted November 4, 2026 · Updated November 5, 2026 · Also on [cyberhippo.ca](https://cyberhippo.ca/writeups/how-i-use-ai)*

## Executive summary

I've been working with AI since 2024. It started with learning how to write good prompts and has grown into automations that run every day, models running on my own hardware, and a habit of checking AI work before I trust it.

- **Tools:** ChatGPT, Claude, Gemini and Microsoft Copilot, plus local models through Ollama
- **Automation:** daily Claude automations that do tasks and update documentation, including my job search and a scam intelligence tracker, some with review checks built in
- **Local AI:** multiple Hermes Agents running home lab tasks, and Stable Diffusion for image generation
- **Quality:** I check AI-written code with web tools and run different AI agents against each other before trusting the result
- **At work:** Microsoft Copilot at KPMG and Gemini at Neo Financial for charts, cross-checking documents and spreadsheets, executive summaries and research, plus n8n automations at Neo that cut manual onboarding steps

## The bigger picture

Everyone's talking about AI, but it's been around for a while now, and it's been a rollercoaster. Some jobs have been lost to it, new ones have been created, and a lot more have changed, along with the way we get everyday stuff and work done.

Technology replacing roles isn't new. It's happened with every big shift, from factory automation to computers to the internet. AI is the latest, and it's moving fast. It keeps pushing the limits of what's possible, but it also brings new problems: answers that sound right but aren't, security and privacy risks, and real questions about who's accountable when it gets something wrong.

## How I use AI day to day

I use AI as a productivity tool and an extra set of hands. It helps me research, troubleshoot problems, write code, analyze information, automate repetitive tasks, brainstorm ideas and break down complicated topics. I also use it to compare options, like A/B tests at work or decisions in my personal life, to see which one the data actually supports. It gets work done much faster than I could on my own, but it isn't always right and it doesn't replace human judgment. I make the decisions, verify the information and take responsibility for the results.

It's also part of everyday life. It helps me come up with creative bedtime stories to read to my 6-year-old, and tweak recipes when I'm missing an ingredient. But it can go a lot deeper than that.

## 1. 2024: Getting started with ChatGPT

I started by learning prompt engineering in ChatGPT. I built my first personas, custom personalizations and agents, including one set up as a cybersecurity mentor to explain concepts and quiz me while I studied.

## 2. Everyday assistant with Gemini

I use Gemini as a day-to-day assistant. I talk through ideas with it hands-free by voice while I drive, and I use it to search my email for upcoming events and to organize my calendar and plan my day.

## 3. Daily automations with Claude

Since then I've moved on to automations with Claude that run every day to do tasks and update documentation. Some have review checks built in and some run on their own. A few examples:

- **Job search:** my own prompts scan job postings through an MCP connector running in Docker. Claude also helps me update my resume and cover letters, and everything is tracked in a spreadsheet in Google Drive.
- **Scam intelligence tracker:** pulls official scam alerts and checks phishing feeds like PhishTank, OpenPhish and URLhaus, and keeps a running record of what's circulating.
- **Following a court case:** a daily check for updates on a case I'm following.

## 4. Running AI locally

- I run multiple Hermes Agents on a Windows PC, each handling a different task in my home lab, using Qwen models through Ollama. The setup behind it is in my [Project NOMAD write-up](project-nomad.md).
- I've been experimenting with image generation using Stable Diffusion running locally on my MacBook Pro.

## 5. Vibe coding, then checking the work

I've built simple websites with AI. I don't take the code at face value. I QA it myself using web tools that check for good coding practice, and I run different AI agents against each other to catch what one of them misses.

## 6. At work

At KPMG, Microsoft Copilot was my office AI and I used it every day. I was wearing a lot of hats, and it helped me automate a lot of the business analyst work: it built charts and graphs, cross-referenced documents against spreadsheet data, turned long documents into executive summaries and sped up research.

At Neo Financial, Gemini filled the same role for summarizing documents, cross-referencing data and research. I also built n8n automations that added new hires to their Slack channels, cutting out manual steps in Jira and Slack.

## What I learned

- **Clear direction matters.** The quality of what comes back depends on how well I describe what I want.
- **Check the work.** AI output gets reviewed, tested or cross-checked by another tool before I rely on it.
- **Stay in control of the data.** I think about what I'm sending to a cloud service, and running models locally keeps some of it on my own hardware.

I don't think the answer is to ignore AI or trust it blindly. The value is in knowing where it helps, checking its work and owning the result.

---

[All write-ups](README.md) · [cyberhippo.ca](https://cyberhippo.ca)
