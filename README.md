# Problem-Statement-3
1) Executive Summary

We propose Sentinel‑Edge, a modular on‑device multi‑agent system that learns each user’s behavioral biometrics and usage patterns locally and performs real‑time anomaly detection with no raw data leaving the device. The system uses lightweight models (TFLite/Core ML) with incremental on‑device personalization and a risk‑fusion policy engine to trigger graduated responses (step‑up auth, session lock, honeypot mode, etc.).

Key goals:

Privacy‑first: all sensitive behavioral signals remain on device; models are personalized locally.

Real‑time: continuous streaming inference (<30 ms/segment targets) with battery‑aware scheduling.

Resilient: detection for unauthorized access, bot‑like automation, replay/spoofing, device‑share drift.

Portable: works on Android (TFLite/LiteRT) and iOS (Core ML), optional Wear OS/watchOS extensions.

2) System Architecture
2.1 High‑level Overview
+----------------------------- Sentinel-Edge Runtime (On Device) -----------------------------+
|                                                                                             |
|  Sensors & Context        Feature Agents         Behavior Models        Risk & Response     |
|  -----------------        --------------        ----------------        ---------------     |
|  - Touch events           - Touch Agent         - Touch HMM/TCN        - Risk Fusion       |
|  - Key timings            - Keystroke Agent     - Keystroke TCN/GRU    - Policy Engine     |
|  - Motion (accel/gyro)    - Motion Agent        - Gait CNN/GRU         - Action Manager    |
|  - App usage/foreground   - App-Usage Agent     - App n-gram Seq2Seq   - Audit & Telemetry |
|  - Device posture/GPS*    - Context Agent       - Context embedding     - On-device logs    |
|                           - Anti‑Spoof Agent    - Liveness classifier   - UI Prompts        |
|                                                                                             |
|                    <----- Shared Ring Buffers + Feature Store + Keychain ----->             |
|                                                                                             |
|                   On‑Device Personalization (LiteRT/Core ML update, EMA/PLR)                |
+---------------------------------------------------------------------------------------------+

* GPS disabled by default; coarse activity inferred from motion to minimize data collection.

2.2 Agent Roles

Touch Agent — captures swipe/scroll histograms, pressure/area (when available), gesture velocity/acceleration, micro‑tremor spectra.

Keystroke Agent — inter‑key latencies (down–down, up–down), hold times, error/auto‑correct rates, keyboard layout context, language model state deltas.

Motion Agent — micro‑gait signatures while typing/scrolling, device‑in‑hand dynamics; segments idle vs locomotion.

App‑Usage Agent — temporal patterns (time‑of‑day, session length), foreground switches, notification interactions.

Context Agent — coarse state: locked/unlocked, charging, network type, biometrics availability. No raw content.

Anti‑Spoof & Bot Agent — detects scripted input (constant latencies, perfect periodicity), replay traces (low jitter), emulator signals.

Risk Fusion — Bayesian/ensemble fusion of per‑agent anomaly scores; produces a single RiskScore ∈ [0,1].

Policy/Response Engine — maps RiskScore + context → actions (soft prompts, step‑up auth, lock, hidden‑value throttle, decoy views/honeypot, local alert).

2.3 Data Flow

Ingestion → Featureization (windowed 3–10 s) → Per‑agent inference → Risk fusion (exponential decay & context priors) → Policy → Action → Secure audit (rolling window, encrypted at rest).

3) Modeling Approach
3.1 Per‑Agent Models (lightweight)

Touch: Temporal Convolutional Network (TCN) or 1D‑CNN on gesture sequences + frequency features; fallback: Gaussian Mixture Models (GMM) / One‑Class SVM for ultra‑low‑end devices.

Keystroke: GRU/TCN on timing vectors; derived n‑gram timing embeddings; fallback: Univariate thresholds + Mahalanobis distance on handcrafted features.

Motion/Gait: CNN‑GRU over accelerometer/gyroscope windows; spectral roll‑off, entropy features.

App‑Usage: PPM/n‑gram or compact transformer for next‑app prediction; anomaly = low next‑token probability.

Anti‑Spoof: Lightweight liveness classifier + rule features (jitter variance, quantization artifacts, emulator flags) → logistic regression.

All models exported as .tflite (Android) or .mlmodel (iOS). Quantization: int8 post‑training where possible; mixed‑precision for timing models.

3.2 Personalization & Continual Learning

Cold‑start: population priors (generic models) with fast personalization via EMA‑adapted centroids and probability‑ratio calibration over first N sessions.

Online updates: periodic prototype rehearsal (small exemplar buffer) + elastic weight consolidation (EWC‑lite) for neural heads to resist drift.

Private training: all gradient steps local; no raw samples leave device. Optional federated averaging of encrypted gradients is disabled by default (can be made opt‑in with DP noise).

3.3 Risk Fusion

Normalize per‑agent anomaly scores with calibrated Platt scaling.

Fuse using Bayesian log‑odds with context priors (time, location coarse state, unlock method).

Temporal smoothing via exponential decay and CUSUM to detect sudden shifts.

4) Privacy, Security & Trust

Data minimization: capture only timings/kinematics, not content; strip key codes, store only positions in coarse bins.

Secure storage: encrypted feature store; model keys in hardware‑backed keystore/secure enclave; bind personalization to device/OS install.

Attack surface & mitigations:

Replay/bot: detect low‑jitter sequences, unnatural periodicity, input automation APIs, emulator heuristics.

Shoulder‑surf / handover: rapid touch‑dynamics divergence + motion signature mismatch triggers step‑up auth.

Poisoning: require trusted context (recent biometric/passcode unlock) for learning; cap per‑epoch updates; outlier‑robust losses.

Model extraction: encrypt model files at rest; integrity tag; runtime attestation checks (root/debug/emulator flags → disable learning & throttle features).

Compliance notes: all processing is on‑device; provide clear consent screens and a privacy switch to pause collection and wipe learned state.

5) UX & Response Policies

Invisible by default; show gentle friction when risk crosses thresholds:

Low: subtle banner, in‑app re‑auth suggestion.

Medium: step‑up (biometric/passcode), rate‑limit sensitive actions, mask balances/PII.

High: session lock, revoke tokens, switch to honeypot/decoy data view, require full login.

Accessibility & transparency: explain why a challenge occurred; provide a privacy dashboard and local data reset.

6) Implementation Plan & Repo Layout
repo/
├── README.md                          # per template root
├── docs/
│   ├── TECHNICAL_SOLUTION.md          # this document
│   ├── ARCHITECTURE.md                # detailed diagrams & flows
│   ├── PRIVACY_SECURITY.md            # threat model & mitigations
│   ├── EVALUATION.md                  # metrics, protocols
│   └── UX_POLICY.md                   # risk thresholds & UI copy
└── src/
    ├── android/
    │   ├── app/
    │   │   ├── java/...               # agents, fusion, policy
    │   │   ├── assets/models/         # .tflite models
    │   │   └── jni/                   # optional native feat.
    │   └── build.gradle
    ├── ios/
    │   ├── SentinelEdge/
    │   │   ├── Agents/                # Swift agents
    │   │   ├── Models/                # .mlmodel
    │   │   └── Policy/
    │   └── Podfile / Package.swift
    └── models/
        ├── base_touch_int8.tflite
        ├── base_keystroke_fp16.tflite
        ├── base_motion_int8.tflite
        └── app_usage_ppm.bin
6.1 Android (TFLite/LiteRT) skeleton

Kotlin services per agent using foreground service only when required; otherwise JobScheduler/WorkManager with battery‑aware cadence.

TFLite Interpreter with GPU/NN delegate if present; fall back to CPU.

Keystore‑backed encryption for feature store; biometric‑gated learning.

6.2 iOS (Core ML) skeleton

Swift Combine pipelines to collect events; background tasks for batching.

MLUpdateTask for on‑device updates; ModelConfiguration with low‑memory options.

7) Feature Engineering (examples)

Touch: dwell time, path curvature, velocity quantiles, micro‑tremor PSD, pressure deltas, two‑finger ratios.

Keystroke: KD, UD, DU latencies, digraph/trigraph stats, error bursts, adaptive WPM baseline, autocorrect/ backspace cadence.

Motion: energy in 0.8–3 Hz band (hand tremor), device‑in‑hand vs table heuristics, orientation transitions.

App‑Usage: session entropy, Markov next‑app prob., unusual foreground at unusual time, notification interaction lag.

8) Evaluation Protocol

Datasets for pretraining/benchmarking (public): HMOG (touch+motion), Aalto (keystroke timing), SHL (locomotion) for context robustness.

User study (opt‑in): synthetic adversary sessions (scripted taps, emulator, different user handover).

Metrics: EER, ROC‑AUC per agent; Time‑to‑Detect (TTD); Battery cost (mWh/hr); False Intervention Rate per day.

Ablations: model vs rule‑only; fusion strategies; quantization modes.

9) Risk & Edge Cases

Shared device / guest mode: quick secondary profile with temporary baseline; auto‑expire.

Cold start: widen thresholds until N trusted sessions; increase friction only for high‑risk actions.

Mode changes (injury, new keyboard): learning rate warm‑up + volatility detector to reduce over‑correction.
