---
title: Scoring- The Dictionarry Way
slug: scoring-dictionarry-way
author: SFusion, Delavicci 
created: 2026/05/19
tags: [wiki, profiles, scoring, Seraphys]
blurb: The construction of a proper profile (according to Seraphys)
featured: true
---

Every score in a Dictionarry profile is part of an interconnected system. The values look arbitrary, but they encode a strict hierarchy where each layer of a release's identity occupies its own order of magnitude. This guide explains why you can't change one number without understanding all of them.

---

# The Magnitude System

A release's total score is the sum of every Custom Format that matches it. Scores are deliberately set at different *orders of magnitude* so that each layer always dominates the layers below it. Think of it like a number where each "digit" represents a different attribute.

| Layer | Range | What it answers |
|-------|-------|-----------------|
| **Resolution + Source** | 120,000 – 465,000 | The coarsest filter. Is this a 1080p Bluray? A 720p WEB-DL? A 2160p encode? This single score determines the release's broad tier. Everything else is tiebreaking within this layer. |
| **Release Group Tier** | 80,000 – 125,000 | Data-driven rankings of specific encode groups. Tier 1 groups produce provably better encodes. Large enough to reorder releases within a resolution, but never large enough to make a 720p release outscore a 1080p one. |
| **Streaming Source** | 1,000 – 6,000 | Which streaming service the WEB-DL came from. AMZN and DSNP typically offer better bitrates. Matters within a tier but can't override group rankings. |
| **Audio Format** | 200 – 1,600 | Audio codec quality: AAC → DD → DD+ → DTS → FLAC → DTS-HD MA → DTS-X. Fine-grained tiebreaker. Can never override which streaming source was used. |
| **WEB-DL Group Tier** | 20 – 100 | Fine-grained ranking among WEB-DL cappers (FLUX, NTb, etc). At this magnitude, it can only break ties when everything else is equal. |
| **Repacks** | 6 – 8 | Is this a repack of a previous release? Lowest priority tiebreaker — only matters when two releases are otherwise identical. |

:::warning

**The core rule:** Each layer's maximum possible score is always smaller than the minimum meaningful difference in the layer above it. This is what makes the system work — and what breaks when you arbitrarily change a score. If you raise an audio score to `50,000`, it can now override release group tiers. If you raise a streaming source score to `200,000`, it can override resolution preferences. **Every number is constrained by every other number.**

:::

---

# How Releases Get Scored

Every Custom Format that matches a release adds (or subtracts) its score. The total determines whether the release is grabbed (`minCustomFormatScore: 20000`) and whether it upgrades the current file. Here are three releases through the **1080p Quality** profile:

## A Tier 1 Bluray encode

`Movie.2024.1080p.BluRay.DTS-HD.HRA.x264-DON`

| Format | Score |
|--------|-------|
| 1080p Bluray | +280,000 |
| 1080p Quality Tier 1 *(DON)* | +125,000 |
| DTS-HD HRA | +700 |
| **Total** | **405,700** |

**Grabbed.** Score exceeds the 20,000 minimum. A Tier 1 Bluray encode — this is what the profile is designed to find.

## A WEB-DL from Amazon

`Movie.2024.1080p.WEB-DL.DD+5.1.AMZN-FLUX`

| Format | Score |
|--------|-------|
| 1080p WEB-DL | +380,000 |
| AMZN *(streaming source)* | +3,000 |
| Dolby Digital+ | +600 |
| WEB-DL Tier 1 *(FLUX)* | +100 |
| **Total** | **383,700** |

**Grabbed.** But scores **lower** than the DON Bluray above (405,700 vs 383,700) despite WEB-DL having a higher base score. **The group tier is what makes the difference** — Tier 1 adds +125,000 which overcomes the 100K gap between Bluray and WEB-DL.

## An x265 release in the wrong profile

`Movie.2024.1080p.BluRay.DTS.x265-SomeGroup`

| Format | Score |
|--------|-------|
| 1080p Bluray | +280,000 |
| DTS | +300 |
| x265 *(codec penalty)* | −999,999 |
| **Total** | **−719,699** |

**Blocked.** The −999,999 penalty is *mathematically impossible to overcome*. The 1080p Quality profile does not want x265 — that's what Efficient and Compact are for.

---

# The Variant System

x265 isn't simply "allowed" or "blocked." There are **five separate Custom Formats** for x265, each with different matching conditions. Different profiles assign different scores to different variants, creating nuanced per-context rules.

| Custom Format | What It Matches | Profile Behavior |
|---------------|-----------------|------------------|
| `x265` | x265 in title AND not 2160p | −999,999 in Quality, −999,999 in Balanced.<br>Blanket block on all non-4K x265. Used by profiles that only want x264. |
| `x265 (Efficient)` | x265 in title AND not 2160p AND not 1080p | −999,999 in Efficient, −999,999 in Compact.<br>Only blocks x265 *below* 1080p. At 1080p, this format **doesn't match**, so x265 is allowed — but only through scored tier groups. |
| `x265 (Bluray)` | x265 in title AND Bluray source AND not 2160p | −400,000 in Remux, −400,000 in 2160p Quality, −400,000 in 2160p Balanced.<br>Soft penalty. Keeps HEVC Remuxes unaffected (they use `HEVC`, not `x265`). |
| `x265 (WEB)` | x265 in title AND not 2160p AND not Bluray | −999,999 in Remux, −999,999 in 2160p Quality.<br>Hard block on x265 WEB sources in profiles that want Bluray quality. |
| `x265 (Remux)` | `HEVC` in title AND `Remux` in title AND not 2160p | −999,999 in Remux.<br>Blocks 1080p HEVC Remuxes. The 1080p Remux profile wants x264 — HEVC is only standard at 4K. |
| `x265 (Missing)` | 2160p AND Bluray AND not Remux AND no codec labeled | −999,999 in 2160p Quality.<br>4K Bluray encode with no codec label? Unverified — block it. |

:::note

**Why this matters for editing:** If you remove the `x265` penalty from the 1080p Quality profile, you'll start getting random x265 encodes from unknown groups. That's not the same as switching to the Efficient profile, which has its own *scored tier system* for x265 groups. Efficient doesn't just "allow" x265 — it only allows x265 from groups with a proven track record, because only tier-listed groups contribute the positive scores needed to cross the 20,000 minimum threshold.

:::

## Same release, different profiles

The same release filename can be a top grab, a fallback, or completely blocked depending on which profile evaluates it.

**`Movie.1080p.BluRay.DDP5.1.x265-QxR` in 1080p Quality:**

| Format | Score |
|--------|-------|
| 1080p Bluray | +280,000 |
| DD+ | +600 |
| x265 *(matches: not 2160p ✓)* | −999,999 |
| **Total** | **−719,399** |

**Blocked.** x265 is unconditionally penalized at 1080p in this profile.

**`Movie.1080p.BluRay.DDP5.1.x265-QxR` in 1080p Efficient:**

| Format | Score |
|--------|-------|
| Efficient Movie Bluray Tier 1 *(QxR)* | +323,000 |
| DD+ | +600 |
| x265 (Efficient) *(not 1080p fails → no match)* | — |
| **Total** | **323,600+** |

**Grabbed.** QxR is a top Efficient group. The `x265 (Efficient)` format doesn't fire at 1080p.

**`Movie.1080p.BluRay.DDP5.1.x265-Vyndros` in 1080p Balanced:**

| Format | Score |
|--------|-------|
| 1080p Bluray | +280,000 |
| DD+ | +600 |
| x265 *(matches: not 2160p ✓)* | −999,999 |
| **Total** | **−719,399** |

**Blocked.** Balanced uses the blanket `x265` CF. Same outcome as Quality — x265 is not welcome.

**`Movie.1080p.BluRay.DDP5.1.x265-Vyndros` in 1080p Compact:**

| Format | Score |
|--------|-------|
| Compact Movie Bluray Tier 1 *(Vyndros)* | +703,000 |
| DD+ | +600 |
| x265 (Efficient) *(not 1080p fails → no match)* | — |
| **Total** | **703,600+** |

**Top tier.** Vyndros is Tier 1 in Compact. Compact has its own tier system specifically for x265 groups — it doesn't just "allow" x265, it *ranks* x265 encoders.

**`Movie.2160p.UHD.BluRay.DDP5.1.DV.HDR10.x265-DON` in 2160p Quality:**

| Format | Score |
|--------|-------|
| UHD Bluray | +420,000 |
| 2160p Quality Tier 1 *(DON)* | +465,000 |
| DV + HDR10 + DD+ | +4,600 |
| **Total** | **889,600** |

**Top tier.** DON is Quality Tier 1. This is the dream release — scored sky-high.

**`Movie.2160p.UHD.BluRay.DDP5.1.DV.HDR10.x265-DON` in 2160p Balanced:**

| Format | Score |
|--------|-------|
| UHD Bluray | +420,000 |
| No matching tier *(DON isn't in Balanced tiers)* | 0 |
| DV + HDR10 + DD+ | +4,600 |
| **Total** | **424,600** |

**Grabbed** — but at 424K, **this loses to any 4K WEB-DL** (440K base). Balanced targets WEB-DLs, so a plain AMZN WEB-DL (~449K with metadata) beats a DON Bluray encode. Same release, opposite philosophy.

---

# Missing Formats Are Not Penalties

Formats like **HDR (Missing)**, **Atmos (Missing)**, and **TrueHD (Missing)** confuse users because they look like negative conditions. They're actually the opposite: **they infer metadata that release groups forgot to put in the filename**. They score positively — they give credit where credit is likely due.

## HDR Missing

`1080p ✓ + Bluray ✓ + Dolby Vision ✓ + x265 ✓ + NOT HDR + NOT HDR10 + NOT SDR`

**The logic:** If a release has Dolby Vision + x265 from a Bluray at 1080p but *doesn't label* any HDR format, it almost certainly has HDR anyway — the group just didn't include it in the filename. So this format scores positively (typically `+1,000`) to give the release credit for HDR it probably has.

## Atmos Missing

`TrueHD ✓ + 7.1 Surround ✓ + NOT Atmos`

**The logic:** TrueHD 7.1 without the Atmos label is almost always Atmos. The format scores the same as regular Atmos (`+400`), ensuring mislabeled releases aren't unfairly ranked below properly-labeled ones.

## x265 Missing

`2160p ✓ + Bluray ✓ + NOT Remux + NOT x264 + NOT x265`

**Different purpose:** Unlike HDR/Atmos Missing, this scores *negatively* (−999,999 in 2160p Quality). A 4K Bluray encode with no codec label is suspicious — if a trusted group encoded it, they'd label it. The "Missing" here means "we can't verify quality, so reject it."

:::note

**The naming pattern:** `(Missing)` always means "the filename doesn't include this label, but based on other evidence..." Whether that inference is positive (HDR Missing → probably has HDR → +score) or negative (x265 Missing → can't verify codec → −score) depends on what the format is trying to protect against.

:::

---

# Why Changing One Score Breaks Everything

Because scores are additive across magnitude layers, changing a value at one layer can cause releases from a lower-priority category to outscore releases from a higher-priority one. Here are concrete examples of how "simple" changes cascade.

## Boosting Atmos

Setting Dolby Atmos to 150,000.

**Before** (Atmos: 400) — Release A: Tier 1 Bluray encode + DTS:

| Format | Score |
|--------|-------|
| 1080p Bluray | 280,000 |
| Quality Tier 1 (DON) | 125,000 |
| DTS | 300 |
| **Total** | **405,300** ✓ |

Release B: Random 1080p WEB-DL + Atmos:

| Format | Score |
|--------|-------|
| 1080p WEB-DL | 380,000 |
| Atmos | 400 |
| **Total** | **380,400** |

Tier 1 encode wins. Group quality outweighs audio.

**After** (Atmos: 150,000) — Release A unchanged at **405,300**. Release B:

| Format | Score |
|--------|-------|
| 1080p WEB-DL | 380,000 |
| Atmos | 150,000 |
| **Total** | **530,000** ✗ |

A random WEB-DL now beats a Tier 1 encode.

:::warning

**What happened:** An unknown group's WEB-DL rip with Atmos (530K) now outscores a DON Tier 1 Bluray encode with DTS (405K). You've told the system that having Atmos matters more than who encoded the release. **The entire quality ranking system is overridden by one audio tag.**

:::

## Banning HDR10+

Setting HDR10+ to −999,999 in 2160p Balanced.

**Before** (HDR10+: +2,000) — 4K WEB-DL from AMZN with DV + HDR10+:

| Format | Score |
|--------|-------|
| 2160p WEB-DL | 440,000 |
| DV | 3,000 |
| HDR10+ | 2,000 |
| AMZN + HDR + DD+ etc | ~4,600 |
| **Total** | **~449,600** ✓ |

Top-quality 4K WEB-DL. Exactly what Balanced wants.

**After** (HDR10+: −999,999) — same release, same metadata:

| Format | Score |
|--------|-------|
| 2160p WEB-DL | 440,000 |
| DV | 3,000 |
| HDR10+ | −999,999 |
| AMZN + HDR + DD+ etc | ~4,600 |
| **Total** | **−552,399** ✗ |

Your best 4K WEB-DLs are now blocked.

:::warning

**The trap:** HDR10+ doesn't exist in isolation. On 4K WEB-DLs, HDR10+ almost always appears *alongside* Dolby Vision and HDR10 — it's part of the premium metadata stack. By blocking HDR10+, you haven't just "removed a preference" — **you've blocked most of the best 4K WEB-DLs that exist.** The releases you actually want are the ones most likely to have HDR10+.

:::

## Custom group preferences

**"Prefer GroupX" (+200,000)** — you create a custom CF for GroupX because you like their encodes:

| Release | Format | Score |
|---------|--------|-------|
| GroupX at 720p WEB-DL | 720p WEB-DL | 240,000 |
| | GroupX bonus | 200,000 |
| | **Total** | **440,000** |

Meanwhile, a Tier 1 1080p Bluray:

| Format | Score |
|--------|-------|
| 1080p Bluray | 280,000 |
| Quality Tier 1 (DON) | 125,000 |
| **Total** | **405,000** |

GroupX's 720p beats a Tier 1 1080p. Your score jumped the resolution barrier.

**"Block GroupY" (−100,000)** — you want to discourage GroupY:

| Format | Score |
|--------|-------|
| 1080p Bluray | 280,000 |
| Quality Tier 2 *(GroupY is tiered)* | 124,000 |
| Your penalty | −100,000 |
| **Total** | **304,000** |

Still 304,000 — well above 20,000. GroupY still gets grabbed. **The tier absorbs your penalty.**

:::warning

**Both directions fail.** Boosting a group with a large positive lets their 720p releases outrank other groups' 1080p. Penalizing a tiered group with a "moderate" negative doesn't work because the tier's positive score absorbs it. **Scores are additive across ALL matching CFs** — your custom penalty and the existing tier both apply and just sum together. To truly block a group, add them to the Banned Groups CF (−999,999). To prefer one, keep the score within the group tier magnitude (~80K–125K) or you'll break the layers above.

:::

---

# The Minimum Score Gate

Every profile sets `minCustomFormatScore: 20000`. A release must accumulate at least 20,000 points from Custom Formats before it can be downloaded at all. This is the final safety net.

:::note

**Why 20,000?** Look at the magnitude layers. Streaming sources max out around 6,000. Audio maxes around 1,600. WEB-DL tiers max at 100. Even if a release matched every single minor positive format, it couldn't reach 20,000 without also matching a **resolution/source format** (minimum ~20,000 for DVD/SD tiers). This ensures no release gets grabbed purely on the strength of minor attributes — it must have at least a baseline resolution classification.

:::

:::warning

**What happens at −999,999:** The nuclear penalty is set to ~1 million negative specifically because the theoretical maximum positive score is well below 900,000. Even if a release matched every single positive format simultaneously, it still can't overcome one hard penalty. This is by design: `−999,999` means "no, absolutely not, under any circumstances." A soft penalty like `−400,000` is different — it says "this is heavily discouraged but a sufficiently excellent release could theoretically overcome it."

:::

---

# The Mental Model

When you look at a Dictionarry profile, think of every release as being scored on a checklist where **items are weighted by importance**:

- **100,000s** — What resolution and source type is it? *(foundational)*
- **10,000s** — Who encoded it, and how well do they encode? *(quality assurance)*
- **1,000s** — Where was it sourced from? *(streaming quality preference)*
- **100s** — What audio format does it have? *(minor tiebreaker)*
- **10s** — Which capture group ripped the WEB-DL? *(final tiebreaker)*

Each question is answered by the sum of matching Custom Formats, and the magnitude gap between layers guarantees that a higher question **always** overrides a lower one.

If you change a score: ask yourself whether the new value respects the layer it belongs to. If an audio score is bigger than a group tier difference, or a streaming source score is bigger than a resolution difference — you've broken the hierarchy, and the profile will make decisions you don't expect.
