---
layout: post
title: "Building a Daily RSS Digest Bot with AI (and Telegram)"
categories: [RSS, python, LLM, telegram, automation]
---


I follow a lot of RSS feeds, *a lot*. News, tech blogs, arXiv papers, python
releases, linux stuff, random Reddit threads, and my newsboat queue fills up faster than I can feasibly read it.

What I really wanted was a curated highlights reel: the important stuff surfaces
automatically, and everything else quietly disappears. My own personal news
editor, minus the salary.

Hence, I tried to build a simple one.

---

## 1. The idea

Daily digest. Give me my top 10 articles, ranked by importance, pushed to Telegram (it was the easiest API to figure out and set up).

The diagram is straightforward:

fetch feeds → parse articles → ask an LLM "what's important?" → summarise top picks → send to Telegram

All done with pure cron (on github actions) and a Telegram bot.

---

## 2. Fetching the feeds

I already had a [newsboat urls file](https://raw.githubusercontent.com/sdysch/dotfiles/refs/heads/master/setups/common/.config/newsboat/urls)
for my [dotfiles](github.com/sdysch/dotfiles/) with sections for different categories.
The script parses that same file, groups feeds by category, and collects recent articles from each.

```python
sections = parse_feeds_file(content)
articles = collect_articles(sections, selected_categories, max_per_feed=10)
```

Each article becomes a dataclass with title, URL, summary, source, and category.
Feedparser handles the XML, httpx fetches everything.

---

## 3. Ranking with LLMs 
I let a [language model decide](https://github.com/sdysch/daily_rss_digest/blob/e927cc135506c8bc96cad346124cb786af52c577/src/ranker.py#L12-L24) with an experimental prompt.
The LLM returns indices in order of importance; no messing with weights (for now).

I also added a second pass where the model writes a one-sentence summary for
each selected article explaining why it matters which makes the Telegram
message readable at a glance instead of dumping raw RSS summaries.

---

## 4. Delivering to Telegram
This was completely new to me, and was surprisingly easy to learn and set up.
Once ranked and summarised, the bot formats everything as an HTML message and sends it to a Telegram chat:

<img src="{{ site.baseurl }}/images/20260702/text.png" alt="" figcaption="" class="center"/>

---

## 5. Running on autopilot (for free)
A [GitHub Actions workflow](https://github.com/sdysch/RSS_automated_digest/blob/main/.github/workflows/daily_digest.yml)
runs daily at 8 AM UTC.
It uses [GitHub Models](https://docs.github.com/en/github-models) for the LLM, which is
free with the built-in GITHUB_TOKEN. 

---

## 6. Things I learned
The actual ranking is not tested; I really wanted a POC to see if I could actually do this in an afternoon (I could). Many further improvements will be needed.
- I did find that the model choice matters, a lot (to be expected). GPT-4o-mini handles ranking well but occasionally
picks different top 10 on re-runs. 
- GitHub Models has an 8k token limit. My first run sent far too many articles to the
LLM and got a 413 error (oops). Truncating to 50 articles and capping summary lengths
fixed it initially. 

---

## 7. Next steps
What I'd add if I had more time:
- Local archive — save a markdown file of each digest so I can `grep` through
what was sent last week
- Per-feed priority hints/weights. Some blogs are must-read, others are nice-to-have.
The LLM doesn't know that yet

The code is on GitHub [sdysch/RSS_automated_digest](https://github.com/sdysch/RSS_automated_digest) if you
want to adapt it for your own feeds. PRs are always welcome, especially if you add a
feature I talked myself out of for now.
