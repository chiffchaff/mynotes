---

## Executive Summary: Recommended Setup

**Primary Choice: M2 Mac mini (16GB) — ~$499 refurbished or base model on sale**

This is the only sub-$500 option that delivers real-time agent responsiveness with acceptable model coverage. NVIDIA workstations at this price point are dead ends; cloud hybrids are your scaling escape hatch.

---

## Hardware Options Analysis ($500 Ceiling)

### Option A: M2 Mac Mini (16GB Unified Memory)
- **Cost:** $499 (Apple refurb) or base 8GB + external challenges
- **Supported Models:** 7B-13B native, 34B-70B quantized (Q4_K_M)
- **Tokens/sec:** 
  - 7B Q4: ~25-35 tok/s
  - 13B Q4: ~15-20 tok/s  
  - 34B Q4: ~8-12 tok/s
  - 70B Q4: ~4-6 tok/s (usable for reasoning, not chatty loops)
- **Power:** 6-8W peak, silent
- **Upgrade Path:** None (soldered) — plan for replacement, not expansion

**Verdict:** Best $500 option. Unified memory bandwidth (~100GB/s) beats entry-level NVIDIA VRAM bandwidth.

### Option B: Used Intel Workstation + Entry NVIDIA Card
- **Cost:** $200-300 used Dell/HP SFF + $200 GTX 1660 Super/Ti (6GB)
- **Supported Models:** 7B-13B FP16, 34B quantized (tight on 6GB)
- **Tokens/sec:** 
  - 7B: ~20-30 tok/s (CUDA)
  - 13B: ~10-15 tok/s
  - 34B Q4: ~5-8 tok/s (VRAM thrashing risk)
- **Power:** 150-200W, fan noise
- **Upgrade Path:** GPU swap possible, PCIe 3.0 limits

**Verdict:** Skip. Used market lottery, power/noise costs, 6GB VRAM is a hard ceiling for agents.

### Option C: Raspberry Pi 5 + External TPU (Experimental)
- **Cost:** $150 Pi 5 8GB + $100 Coral USB + enclosure
- **Supported Models:** 3B-7B int8 only (llama.cpp ARM NEON)
- **Tokens/sec:** 7B Q4: ~3-5 tok/s
- **Verdict:** Novelty only. Too slow for agent loops.

---

## Apple Silicon vs CUDA: Mandatory Analysis

| Factor | Apple Silicon (M2) | Entry CUDA (GTX 1660S) |
|--------|-------------------|------------------------|
| **Memory Bandwidth** | ~100 GB/s unified | 336 GB/s (but 6GB trapped) |
| **Effective Model Range** | 7B-70B Q4 | 7B-13B FP16, 34B marginal |
| **llama.cpp Perf** | Optimized Metal backend | CUDA backend standard |
| **vLLM Compatibility** | ❌ Not supported | ✅ Full support |
| **Quantized Performance** | Excellent (Q4_K_M default) | Good, VRAM-bound |
| **Upgrade Flexibility** | None | GPU swap possible |
| **Power/Noise** | 6W, silent | 125W, audible |
| **Future-Proofing** | Replace in 2-3yr | GPU → better GPU |

**Critical Insight:** At $500, Apple Silicon wins on *usable memory capacity*. A 16GB M2 runs 70B Q4 comfortably; a $200 GPU hits a 6-8GB wall. CUDA only wins if you can spend $800+ for RTX 3060 12GB or $1200+ for RTX 4070.

---

## Quantization Strategy for OpenClaw

### Recommended Formats

| Use Case | Format | Size Mult. | Quality | Speed |
|----------|--------|-----------|---------|-------|
| **Coding assistants (7-13B)** | Q4_K_M GGUF | 4.5GB | 95% | Fast |
| **Reasoning models (34-70B)** | Q4_K_M GGUF | 22-40GB | 90% | Moderate |
| **Long-context agents** | Q5_K_S | +20% mem | 93% | Slightly slower |

### Deployment Priority for OpenClaw + Ollama:
1. **Primary:** Q4_K_M (llama.cpp) — Ollama default, best speed/quality
2. **Fallback:** Q5_K_M for 70B reasoning when quality > speed
3. **Avoid:** Q2_K, Q3_K_L (degrades agent tool-calling accuracy)

**Memory Math (M2 16GB):**
- System + OpenClaw: ~3GB
- Available for models: ~12GB
  - Fits: 13B Q4 + 7B Q4 simultaneously (good hybrid)
  - Fits: 34B Q4 solo (daily driver)
  - Fits: 70B Q4 solo (reasoning tasks, swap when needed)

---

## Cloud GPU Comparison (Hybrid Escape Hatch)

| Provider | GPU | VRAM | Cost/Hr | Latency to US |
|----------|-----|------|---------|---------------|
| RunPod | RTX 3090 | 24GB | $0.44 | 20-40ms |
| Lambda | A10 | 24GB | $0.60 | 15-25ms |
| Vast.ai | RTX 4090 | 24GB | $0.35-0.60 | Variable |
| Self-vLLM | A100 40GB | 40GB | $2.50+ | - |

### Monthly Cost Estimates (Moderate Usage):
- **Light:** 2hr/day × 30 × $0.50 = **$30/mo**
- **Moderate:** 4hr/day × 30 × $0.50 = **$60/mo**
- **Heavy:** 8hr/day × 30 × $0.50 = **$120/mo**

### Hybrid Suitability:
✅ **Recommended pattern:** Local 7-13B for fast tool-calling loops, cloud 70B for deep reasoning. Local handles 90% of calls; cloud on-demand for complex chains.

---

## Deployment Topology Recommendation

### Recommended: Split Architecture (Hybrid Tiered)

```
┌─────────────────────────────────────────┐
│         OpenClaw (Local)                │
│              Mac mini M2                │
│  ┌──────────────┐    ┌──────────────┐ │
│  │  13B Coder   │◄──►│  34B General │ │  (local inference)
│  │  (fast loop) │    │  (reasoning) │ │
│  └──────────────┘    └──────────────┘ │
│           │                             │
│           ▼ (on-demand)                 │
│    ┌──────────────┐                     │
│    │  70B Remote  │◄────────────────────┐ │
│    │  (RunPod)    │   $0.44/hr         │ │
│    └──────────────┘                    │ │
└─────────────────────────────────────────┘ │
                                              │
┌─────────────────────────────────────────┐   │
│    Cloud Inference (vLLM/Ollama)        │◄──┘
│    RTX 3090 / 24GB VRAM                 │
│    Deep reasoning, long context         │
└─────────────────────────────────────────┘
```

### Decision Logic:
- **Tool-calling loops:** Local 13B (<100ms response) — **Real-time usable**
- **Code completion:** Local 13B-34B depending on complexity — **Usable with friction** for 34B
- **Multi-step reasoning:** Local 34B if patient; Remote 70B if urgent — **Background-task viable** local, **Real-time** remote
- **Research summarization:** Remote 70B — **Real-time usable**

---

## Model Tier Strategy

### Tier A: Small Coding Assistants (7-13B)
- **Target:** Qwen2.5-Coder-7B-Instruct, DeepSeek-Coder-V2-Lite
- **Deployment:** Local M2 (mandatory)
- **Responsiveness:** ~20-35 tok/s — real-time usable
- **Use:** Tool-calling, quick completions, chat

### Tier B: Daily-Driver Reasoning (30-34B)
- **Target:** Qwen2.5-32B-Instruct, DeepSeek-V2.5-32B (distilled)
- **Deployment:** Local M2 (tight but works at Q4)
- **Responsiveness:** ~8-12 tok/s — usable with friction for interactive, background-task viable for generation
- **Use:** Complex analysis, multi-step plans, code review

### Tier C: Heavy Reasoning (70B+)
- **Target:** DeepSeek-R1-Distill-Llama-70B, Qwen2.5-72B
- **Deployment:** Remote (local at Q4 runs ~4-6 tok/s — borderline)
- **Responsiveness:** Remote ~20-40 tok/s — real-time usable
- **Use:** Deep research, complex reasoning chains

---

## Cost Strategy Comparison Table

| Metric | M2 Mac mini 16GB (Refurb) | Cloud-Only (RunPod) | Used Workstation + GTX 1660S |
|--------|---------------------------|---------------------|------------------------------|
| **Upfront** | $499 | $0 | $450 |
| **Monthly** | $0 | $30-120 | $15 (power) |
| **Year 1 TCO** | **$499** | $360-1440 | $630 |
| **Latency** | 0ms | 20-40ms | 0ms |
| **Privacy** | ✅ Full | ❌ Cloud sees it | ✅ Full |
| **Maintenance** | None | Provider-managed | Driver hell |
| **Perf Ceiling** | 70B Q4 @ 4-6t/s | Unlimited (pay) | 13B @ 15t/s |
| **Agent Real-Time** | ⚠️ 7-13B yes, 34B ok | ✅ Yes | ⚠️ 7B yes, 13B ok |
| **OpenClaw Quality** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |

---

## Lifecycle Planning (12-Month Evolution to Always-On)

### Month 0-6: Baseline M2 Mac mini
- **Workload:** Local 7-13B primary, 34B for heavy tasks
- **Pattern:** Daily driver, shut down overnight
- **Cloud supplement:** $20-40/mo for 70B needs

### Month 6-12: Reliability Hardening
- **Add:** UPS ($80), network hardening
- **Shift:** Keep 7-13B local, move 34B+ to self-hosted remote

### Month 12-18: Scale Out
- **Decision Point:** If agent usage is 24/7, consider:
  - **Path A:** Upgrade to M4 Mac mini 24GB (~$800) — sell M2 (~$300) = $500 net
  - **Path B:** Build dedicated Mini-ITX + RTX 4060 Ti 16GB (~$1000) — keeps M2 as failover
  - **Path C:** Stable cloud hybrid at $60/mo = $720/yr (comparable to hardware depreciation)

### Component Priority:
1. **First:** Mac mini 16GB (non-negotiable baseline)
2. **Deferred:** External GPU, more RAM (impossible on M2), secondary workstation

---

## Final Recommendation Architecture

### Under $500: M2 Mac Mini 16GB (Refurbished)

**Immediate Deployable Setup:**
- **Hardware:** M2 Mac mini 16GB unified memory — $499
- **Local Models:**
  - Qwen2.5-Coder-7B (Q4_K_M) — tool-calling, fast loops
  - Qwen2.5-14B-Instruct (Q4_K_M) — daily driver
  - DeepSeek-V2.5-32B (Q4_K_M) — heavy local reasoning (swappable)
- **Remote Fallback:** RunPod RTX 3090 for 70B or heavy concurrency
- **Quantization:** Q4_K_M GGUF standard, Q5_K_M for critical reasoning tasks
- **Orchestration:** OpenClaw local, Ollama backend, cloud bridge configured

**Expected Responsiveness:**
- Tool-calling: **Real-time** (~25-35 tok/s, 7B)
- Coding iteration: **Real-time** (~15-20 tok/s, 13B) 
- Reasoning chains: **Usable with friction** (~8-12 tok/s, 34B) or **background** for 70B local
- Hybrid 70B remote: **Real-time** (~25-35 tok/s)

**12-18 Month Roadmap:**
- Month 6: Add $30/mo cloud budget for seamless 70B access
- Month 12: Evaluate M4 24GB upgrade (~$500 net after resale) or stay hybrid
- Month 18: If usage justifies, add dedicated Linux + RTX 4060 Ti 16GB node (~$1000) for CUDA ecosystem access

**Why this wins:** It's the only $500 path that runs 34B locally (real complexity threshold) and 70B passably, with zero operational overhead. NVIDIA at this price is a VRAM trap. Cloud-only burns budget fast. Hybrid keeps you operational today with a credible upgrade path.

Deploy the M2. Start with 7-13B local, add cloud for 70B heavy lifting. Reassess at 12 months.
