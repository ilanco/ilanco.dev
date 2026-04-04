---
title: 'Cronios Landing Page'
date: 2026-04-03T22:09:23+03:00
tags:
  - "Cronios"
  - "saas"
  - "build-in-public"
params:
  author: Ilan Cohen
---

# Cronios Landing Page: Validating Before Building

Usually, I start coding right away and don't think about whether what I'm building has a need. This time, I followed the advice we've all read so many times before and ignored: validate your idea before writing a single line of code.

So, I did. For [Cronios](https://cronios.io/), a Cron Job Monitoring SaaS, I created a landing page.

## Why a Landing Page First?

The idea behind Cronios is simple: monitor your cron jobs and get alerted when they fail. It's a pain point I've experienced firsthand. Silent cron failures that only get discovered when something breaks downstream. But just because I've felt the pain doesn't mean others will pay for a solution.

Instead of building the entire product, I decided to test demand with a single-page landing site. The goal was to describe the product, show what it would look like and see if developers would sign up for early access.

## The Tech Stack

I went with:

- **Next.js 16** with the App Router
- **React 19**
- **Tailwind CSS v4**
- **shadcn/ui**
- **Geist** and **Geist Mono** fonts
- **Resend** for the email pipeline

All hosted on Vercel.

One detail I'm particularly happy with: the entire design uses **OKLCH** (a new way to encode colors like hex, RGBA, or HSL). The primary color is `oklch(0.55 0.18 250)`, a clean blue.

## Building Without Screenshots

Since the product doesn't exist yet, I couldn't use screenshots. Instead, I built a **mock dashboard preview** entirely in HTML and CSS right in the hero section. It shows three sample cron jobs — a database backup, email queue, and report generator. It has status indicators, cron expressions and last-run times.

The "How it Works" section features a **styled macOS terminal window** showing the single `curl` command needed to integrate with Cronios. Again, pure HTML/CSS.

## The Architecture

The site is a single page with hash-based navigation between sections:

1. **Header**: sticky nav with glassmorphism (`backdrop-blur-lg`), anchor links, and a "Coming Soon" badge
2. **Hero**: headline, email signup form, and the mock dashboard
3. **Features**: 6-card grid with hover effects on icon backgrounds
4. **Stats**: inverted color section with key metrics (99.99% uptime, 10s alert latency, 60s setup)
5. **How it Works**: 3-step walkthrough with connector lines and the terminal code example
6. **CTA**: final conversion section with a second email signup form
7. **Footer**: minimal, with contact info and copyright

Both the Hero and CTA contain identical email capture logic. A common landing page pattern to maximize conversions. The form POSTs to a Next.js API route (`/api/send`) which uses Resend to send a notification email.

## What I Learned

### Validation Works

The landing page approach forced me to present the value proposition clearly before writing any code. If I can't explain why someone should care on a single page, building the product won't fix that.

### Keep It Simple

The entire site is one page. No blog, no docs. Just a clear message and a call to action.

## What's Next

The landing page is live and collecting signups. The next step is to build the actual product, if demand is there.

But for now, I'm glad I resisted the urge to start coding. Validate first, code second.

---

_This post is part of the [Cronios](https://cronios.io/) building-in-public series. Follow along as I build a cron job monitoring SaaS from scratch._

