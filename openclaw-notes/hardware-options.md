# OpenClaw Hardware Options & Recommendations

*Compiled from the OpenClaw Discord #hardware channel (~2,230 messages) — February 6-8, 2026*

---

## TL;DR

| Hardware | Price Range | RAM/VRAM | Power Draw | OS | Best For |
|----------|------------|----------|------------|-----|----------|
| **Mac Mini M4 (base)** | $399-$599 | 16GB unified | 40-60W | macOS (native) | Budget entry, great perf/$ |
| **Mac Mini M4 Pro** | $1,399+ | 24-48GB unified | 40-60W | macOS (native) | Mid-range, better memory bandwidth |
| **Mac Studio M4 Max/Ultra** | $2,000-$11,500 | 64-512GB unified | 75-150W | macOS (native) | Serious local model inference |
| **AMD Strix Halo (Ryzen AI Max 395)** | $2,700-$4,000 | Up to 128GB unified | ~65W | Linux (Ubuntu) | Local model inference, best non-Apple |
| **Beelink SER (AMD Ryzen)** | $300-$800 | Up to 64GB | ~45W | Linux (Ubuntu/Debian) | Non-Apple gateway + light inference |
| **Dell Optiplex (used)** | ~$100 | 8-32GB | ~35W | Linux (Ubuntu/xubuntu) | Ultra-budget gateway |
| **Raspberry Pi 4/5** | $75-$120 | 4-16GB | ~5W | Raspberry Pi OS / Ubuntu | Ultra-budget, gateway-only |
| **NVIDIA Jetson Orin Nano Super** | ~$250 | 8GB + 67 TOPS | ~20W | JetPack (Ubuntu-based) | Always-on, low power, ML tasks |
| **RTX 5090 PC** | $2,000+ | 32GB VRAM | ~450W | Linux or Windows | Fast inference, limited VRAM |
| **Dual NVIDIA 3090s** | $1,500-$2,500 | 48GB VRAM total | ~700W | Linux (Ubuntu) | vLLM serving, max throughput |
| **Synology NAS (Docker)** | Varies | Varies | Low | DSM (Docker) | Gateway-only, use existing hardware |
| **OpenClawPi (pre-built)** | TBD (on sale) | Pi-based | ~5W | Pre-configured | Zero-setup voice assistant |
| **VPS (cloud)** | $5-50/mo | Varies | N/A | Ubuntu Server | Remote, always-on, no hardware |

---

## Operating System Guide

**This is critical — OpenClaw runs on anything with Node.js, but your OS choice matters for local models and ease of setup.**

### macOS (Recommended for most users)
- **Native first-class support** — OpenClaw was built Mac-first
- Desktop app available (easiest install)
- Unified memory = VRAM for local models via Ollama/LM Studio
- Works on: Mac Mini, Mac Studio, MacBook Pro/Air
- **No setup friction** — just install and go

### Linux (Best for non-Apple hardware)
- **Ubuntu 24.04 LTS** is the community default for Beelink, Dell, custom builds
- **xubuntu 24.04** recommended for older/weaker hardware (lighter desktop)
- **Raspberry Pi OS** or **Ubuntu Server** for Pi devices
- **JetPack OS** (Ubuntu-based) for NVIDIA Jetson
- NVIDIA GPU users: Linux is strongly preferred over Windows for vLLM, Ollama, CUDA
- Docker support is excellent on Linux

### Windows
- Works but **not recommended** by the community
- WSL2 is an option but adds complexity
- NVIDIA GPU users report better results on Linux
- If you have an "old Windows PC" — install Linux on it instead

### Docker
- Works on NAS devices (Synology), Linux servers, VPS
- Some skill installation issues reported — mount volumes for persistence
- Good for headless/remote deployments

**Community consensus:** If it's not a Mac, put Ubuntu on it.

---

## Detailed Analysis

### 1. Apple Mac Mini M4

**The community favorite for getting started.**

- Base M4 found for **$399** at Micro Center (Best Buy price match) — normally $599
- 16GB unified memory, 256GB SSD at base config
- M4 Pro models offer significantly better memory bandwidth (important for local models)
- Unified memory = all RAM is available as VRAM

**RAM tiers matter:**
- **16GB** — Good for gateway + cloud API models. Too small for meaningful local inference
- **24GB (M4 Pro)** — Can run smaller models (GLM-4.7-flash, Qwen 14B)
- **32GB (M4 Pro)** — Sweet spot for running GLM-4.7-Flash and similar mid-size models
- **48GB+ (M4 Pro Max)** — Can run 30B+ models with decent context

> *"M4 Pro will be significantly faster compared to the standard M4 due to memory bandwidth"* — Delerious

> *"GLM-4.7-Flash is the current goto, I believe you need a 32gb+ model to run it smoothly."* — Delerious

> *"Mac Mini at $399 is honestly great value for OpenClaw."* — Yanko Aleksandrov

**Minimum for local models:** 24GB (M4 Pro). For gateway-only with API: 16GB is fine.

---

### 2. Apple Mac Studio (M4 Max / M3 Ultra)

**The serious local inference machine. Apple's secret weapon.**

- **M4 Max:** 64-128GB unified memory, 16-core Neural Engine, 40-core GPU
- **M3/M4 Ultra:** Up to **512GB unified memory** — the holy grail for local LLMs
- 512GB config runs ~$11,500

**Why 512GB matters:**
> *"OpenClaw is only useful if the main agent is a huge model (Kimi K2.5 q4 or larger). Which means you need significant vram or unified memory... The only realistic way is to use a Local LLM in something like vLLM or LM Studio... In order to host a great model locally at high tokens per second per watt, you need the 512GB unified memory at 11.5k."* — omnotion

**Recommended Studio config (from SeñorCarlos):**
- **Chip:** M4 Max (16-core CPU, 40-core GPU, 16-core Neural Engine)
- **Memory:** 64GB or 128GB unified
- **Storage:** 2TB SSD (model weights, vector indexes, Docker images)
- **Networking:** 10Gb Ethernet for NAS/node connectivity

**Real-world reports:**
- CiNcRoN: M2 Ultra 128GB running qwen3-coder at 256K context — struggles at 256K, had to scale to 32K
- anisloptera: Regrets not getting 96GB on M2 Ultra Studio

> *"Those $10k Studios have 512GB of vram (effectively). The sixth best model is what, kimi2.5? Minimax? I get reasonable results from glm-4.7-flash."* — anisloptera

---

### 3. AMD Strix Halo (Ryzen AI Max 395)

**The power-user recommendation for local AI inference without Apple.**

- **128GB unified VRAM** — competitive with Mac Studio at lower price
- Available on Amazon for **$2,700-$4,000**
- Available as: Framework Desktop, Beelink GTR9, GMKTek Evo X2
- Excellent tok/sec on models up to 80B
- **OS: Linux (Ubuntu 24.04 recommended)**

**Limitations noted by community:**
> *"The main issue is the current networking on the strix halos doesn't scale and is limited by networking bandwidth between devices (10-50GB ethernet or InfiniBand)... To get the same 512GB vram you're spending just as much in hardware, way more in engineering time, and tanking your token generation across the nodes. Plus 4 of the 128GB APUs are going to pull more energy than 1 of the 512GB Studios."* — omnotion

**Bottom line:** Single best non-Apple option. Don't try to cluster multiple — just buy one with max RAM.

---

### 4. NVIDIA RTX 5090

**Fast but VRAM-limited. Community is divided.**

- 32GB VRAM — sounds like a lot, isn't enough for agentic LLMs
- Can run Qwen 32B, DeepSeek 32B, maybe DeepSeek 70B at 3-bit quant
- **OS: Linux strongly recommended** (better CUDA/vLLM support than Windows)

**The honest take from RTX 5090 owners:**
> *"Can someone please confirm that I'm either dumb or not insane? I have an RTX 5090... Simply put, there is no local model that comes anywhere close to the top 5 cloud models, maybe not even the top 10... I'm having a very hard time understanding why people are buying $10k Mac Studios just to run the ~6th best model."* — heytommyg

> *"I have had a lot of problems with local models on an RTX 5090. I can't find a good model that fits into 32GB VRAM and is also good at agentic tool calling."* — heytommyg

> *"Consumer gaming GPUs are not capable of running LLMs that can do this. You simply do not have the vram to run a model big enough with a big enough context. The break point is around 64GB."* — anisloptera

> *"Maybe if you had two 5090s."* — anisloptera

---

### 5. Dual NVIDIA 3090 Setup

**The local inference powerhouse for Linux enthusiasts.**

- 48GB total VRAM (24GB × 2)
- Can run devistral 2 small with W4A16 GS64 quant at 60 tok/sec
- 170k fp16 context via vLLM
- **OS: Linux (Ubuntu) required** — vLLM doesn't work well on Windows

> *"A coherent agent model such as devistral 2 small with a W4A16 GS64 quant +EoRA is perfectly capable of running this with 170k fp16 context via vLLM on 2 3090s at 60t/s but it requires prompting that isn't absolute garbage"* — Hascrack

---

### 6. NVIDIA Workstation GPUs (A2000, A4000, A4500)

**Budget pro GPUs with decent VRAM.**

- A4000: 16GB, A4500: 20GB, A2000: 6-12GB
- phidauex running GLM-4.7-flash on A2000 + A4000 combo with 64K context, 23GB VRAM total
- **OS: Linux** (Ubuntu + CUDA)
- Ollama pre-release 0.15.5 significantly improved VRAM efficiency

> *"The big tip at the moment is that the current 0.15.5 pre-release of ollama is running glm with a lot less VRAM than before — with the previous version I was running out of VRAM at 32k context, but with the pre-release I can run 64k with room to spare."* — phidauex

---

### 7. Beelink Mini PCs

**Solid non-Apple option for gateway + light local work.**

- Beelink SER series (AMD Ryzen) — good single-thread perf
- Beelink GTR9 — supports Strix Halo with 128GB RAM
- **OS: Install Linux (Ubuntu 24.04)** — they ship with Windows but community recommends switching

> *"Beelinks are solid for the price! ... If you just need a general-purpose box though, Beelink SER series are probably your best bet. The AMD Ryzen ones have good single-thread perf which matters for OpenClaw."* — Yanko Aleksandrov

---

### 8. Dell Optiplex (Used)

**The $100 gateway solution.**

- Dell Optiplex 7060 Micro confirmed working well
- Pair with a cloud API coding plan (Kimi Code, etc.) for a dirt-cheap setup
- **OS: Linux (Ubuntu or xubuntu 24.04)**

> *"Bought a used Dell Optiplex 7060 Micro. Runs OpenClaw really well. I combine this with an existing coding plan. Makes for a very affordable setup. Used Dell Optiplex can be bought for as little as $100."* — tibsmagee

---

### 9. Raspberry Pi 4/5

**Ultra-budget gateway. Don't try to run local models on it.**

- Pi 4 and Pi 5 both confirmed working for gateway
- Pi 5 with 16GB asks: "Is it capable of running local model?" — Answer: **No, not really**
- Great as "mission control" paired with API keys
- **OS: Raspberry Pi OS or Ubuntu Server**

> *"Definitely not. Don't run a local model on it lol. It's fantastic as mission control, not for compute. Slap an anthropic API key on it, ssh in and you're good to go."* — heytommyg

> *"I run it on a pi4 b, so yea"* — blasto (confirming gateway works on Pi 4)

Running local llama on Pi 5: *"it sounds like a mini turbine is running with the cooling system"* — Us+AI

---

### 10. NVIDIA Jetson Orin Nano Super

**Purpose-built edge AI box.**

- 67 TOPS AI compute, 8GB RAM, 512GB NVMe, ~20W
- **OS: JetPack (Ubuntu-based, NVIDIA maintained)**
- Docker-friendly with some ARM gotchas (UID permissions)
- Daily API cost with Sonnet: ~$2-5

> *"Always-on dedicated box for Telegram, browser automation, proactive alerts. Low power draw is perfect for 24/7."* — Yanko Aleksandrov

---

### 11. Old MacBook Pro (10+ years old)

**It can work if you install Linux.**

> *"Your best bet is to install a low weight Linux kernel on that 10 year old MacBook Pro. Try xubuntu 24.04."* — Spartan

---

### 12. VPS / Cloud

**For remote always-on without buying hardware.**

- Archit Mittal's use case: browser scraping, API access, sensitive data (Slack, emails)
- Community split: some say Mac Mini is better for security, others say VPS is fine
- **OS: Ubuntu Server 22.04/24.04**

> *"The limitation won't be hardware. It'll be security architecture discipline."* — SeñorCarlos

---

### 13. OpenClawPi (Pre-built Product)

**Zero-setup Pi + Voice AI + OpenClaw.**

- By RunAnywhere AI (openclawpi.com)
- Pre-configured with local wake word detection
- USB mic + speaker included
- Web console for configuration
- **OS: Pre-configured (Raspberry Pi OS based)**

---

## The Great Local vs Cloud Debate

This is the biggest argument in the channel. Here's the honest breakdown:

### Why people go local:
1. **Privacy** — all data stays on your machine
2. **Cost over time** — no per-token API charges
3. **Independence** — no rate limits, no API outages
4. **Future-proofing** — open models improve; hardware you buy today runs better models tomorrow

> *"I run local models for privacy reasons and because I'm a pennypincher. Local models can do most things I'd pay for a lot if I used cloud models."* — Bastrabun

> *"Keep in mind that the value of the tokens we can generate using this hardware will only go up over time. Open models get better all the time... if your benchmark is today's cloud models then six months from now you can probably see that quality in a local model... if you can run it."* — anisloptera

### Why people go cloud:
1. **Quality gap** — cloud models (Opus 4.6, Codex 5.2) are still significantly better
2. **Agentic tool calling** — local models struggle with complex tool use
3. **Cost efficiency** — $2-5/day API vs $2,700-$11,500 hardware
4. **Simplicity** — no GPU debugging, no quantization headaches

### The hybrid approach (community consensus):
Use cheap hardware (Pi, Mac Mini, Jetson) as the gateway. Run frontier cloud models (Sonnet, Opus) for the main agent. Optionally run local models for sub-tasks, privacy-sensitive work, or as a cost-saving "reviewer" model.

---

## Architecture Discussion: Single vs Multi-Device

From heytommyg and SeñorCarlos:

1. **1 device / 1 install / multiple agents** — Standard for most people
2. **1 device / multiple installs** — Niche, for security-conscious power users
3. **Multiple devices / 1 install** — "Home AI infrastructure" — the future

> *"Option 3 becomes normal once people treat OpenClaw like 'home AI infrastructure.' Partitioning starts to make sense if... agents can execute arbitrary code without strong containment, you are mixing high trust and low trust workflows. That's a clear blast radius problem."* — SeñorCarlos

---

## Local Model Recommendations (from the channel)

| Model | VRAM Needed | Context | Notes |
|-------|------------|---------|-------|
| **GLM-4.7-Flash** | 23-32GB | 32-64K | Current community favorite, good at agentic tasks |
| **Qwen3-Coder** | 128GB | 32-256K | Struggles at 256K, works at 32K on M2 Ultra |
| **DeepSeek 32B** | 20-32GB | Varies | Decent on RTX cards |
| **DeepSeek 70B (Q3)** | 32GB+ | Limited | Heavy quantization hurts quality |
| **Devistral 2 Small** | 48GB (2×3090) | 170K fp16 | Requires vLLM + custom quant |
| **NVIDIA Nemo Nano 30B** | ~20GB | Varies | Untested by community |
| **Nemotron-3** | ~20GB | Varies | "Underwhelming" results reported |

**Key tools:** Ollama, LM Studio, vLLM
**Key tip:** Ollama 0.15.5 pre-release significantly reduces VRAM usage

---

## Memory Requirements Summary

| Use Case | Minimum RAM | Recommended | Why |
|----------|------------|-------------|-----|
| **Gateway only (API models)** | 4GB | 8GB+ | Node.js + browser automation |
| **Gateway + small local model** | 24GB | 32GB | Model weights + system overhead |
| **Serious local inference** | 64GB | 96-128GB | 70B+ models with decent context |
| **Run everything locally** | 128GB | 512GB | Multiple models, large context windows |

> *"You need just slightly more than 64GB to get capable models with acceptable context into memory."* — anisloptera

> *"You really want a context window somewhere between 128K to 200K."* — Naelith

> *"Memory requirements are only going 1 direction with AI. Up."* — omnotion

---

## Guv's Recommendation Matrix

| Use Case | Hardware | OS | Budget |
|----------|---------|-----|--------|
| **Just getting started** | Mac Mini M4 16GB | macOS | $399-$599 |
| **Getting started + some local models** | Mac Mini M4 Pro 32GB | macOS | ~$1,600 |
| **24/7 always-on assistant** | Jetson Orin Nano or Pi 4/5 + API | JetPack / Pi OS | ~$75-$250 + $2-5/day |
| **Serious local inference (Apple)** | Mac Studio M4 Max 128GB | macOS | ~$4,000-$5,000 |
| **Serious local inference (non-Apple)** | AMD Strix Halo 128GB | Ubuntu 24.04 | $2,700-$4,000 |
| **Maximum local everything** | Mac Studio Ultra 512GB | macOS | ~$11,500 |
| **Ultra-budget gateway** | Dell Optiplex (used) or Pi 4 | Ubuntu/xubuntu | $75-$100 + API |
| **Non-Apple power user** | Dual 3090 + vLLM | Ubuntu 24.04 | $1,500-$2,500 + power |
| **Security-sensitive remote** | VPS + API models | Ubuntu Server | $10-50/mo + API |
| **Zero-setup voice assistant** | OpenClawPi pre-built | Pre-configured | TBD |

---

## Interesting Hardware Mentions

- **omi.me** — Wearable AI hardware companion (mentioned by Nate the Plumber)
- **reBot-DevArm** — Robotic arm with open hardware blueprints for embodied AI (shared by AntDX316)
- **Ollama blog** — Official OpenClaw integration guide: https://ollama.com/blog/openclaw

---

*Source: OpenClaw Discord #hardware channel (~2,230 messages), Feb 6-8, 2026*
*Last updated: February 8, 2026*
