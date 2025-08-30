# Problem-Statement-3
🛡️ Sentinel-Edge: On-Device Multi-Agent Security System

Proposal:
A modular, on-device multi-agent system that learns user behavioral biometrics & usage patterns locally 📱 and performs real-time anomaly detection ⚡ with zero raw data leaving the device 🌐❌.

Lightweight ML Models 🤖: TFLite (Android) / Core ML (iOS).

On-device personalization 🔄 with incremental updates.

Risk-fusion engine 🔐: adaptive responses → step-up auth 🔑, session lock ⛔, honeypot mode 🎯.

🎯 Key Goals

🔒 Privacy-First: All sensitive behavioral signals remain on-device; models adapt locally.

⚡ Real-Time: Continuous inference <30 ms/segment, battery-aware 🔋 scheduling.

🛡️ Resilient: Detects unauthorized access 🚫, bot-like automation 🤖, replay/spoofing 🎭, device-share drift 🔄.

📲 Portable: Cross-platform → Android (TFLite/LiteRT) & iOS (Core ML) with WearOS/watchOS ⌚ extensions.

2) System Architecture
   
2.1 High‑level Overview
<img width="1263" height="523" alt="image" src="https://github.com/user-attachments/assets/dd56673b-0e4b-4765-8400-3289acb8274b" />



🛡️ Sentinel-Edge: Modular On-Device Multi-Agent Security
🌍 Privacy by Design

GPS off by default 📴 → only coarse activity from motion.

Minimal signals only ⏱️📈 (timings, kinematics, state).

All data local 🔒 — no raw data leaves device.

🤖 Agent Roles

👆 Touch Agent → swipe/scroll histograms, velocity, micro-tremors.

⌨️ Keystroke Agent → inter-key latencies, hold times, error rates.

📱 Motion Agent → gait signatures, device-in-hand dynamics, idle vs moving.

📊 App-Usage Agent → time-of-day, session length, switch patterns.

🔑 Context Agent → lock state, charging, network, biometrics availability.

🕵️ Anti-Spoof/Bot Agent → detects automation, replay, emulator use.

🧮 Risk Fusion → Bayesian/ensemble anomaly fusion → RiskScore ∈ [0,1].

⚡ Policy Engine → maps RiskScore + context → graduated actions.

🔄 Data Flow

Ingestion 📥 → Featureization (3–10s) ⏱️ → Agent inference 🤖 → Risk fusion 🧮 → Policy → Action 🚨 → Secure audit 🔐.

🧠 Modeling Approach

Touch: TCN / 1D-CNN, fallback GMM/SVM.

Keystroke: GRU/TCN embeddings; fallback thresholds + Mahalanobis.

Motion: CNN-GRU on accelerometer/gyro; spectral features.

App-Usage: n-gram/compact transformer; anomaly = low next-token probability.

Anti-Spoof: liveness classifier + rules → logistic regression.

Deployment: .tflite (Android) / .mlmodel (iOS), int8 quantization ⚡.

🔄 Personalization & Continual Learning

Cold-start: population priors → fast local adaptation.

Online: prototype rehearsal + EWC-lite to resist drift.

Private training: all updates on-device; federated mode opt-in only.

🧮 Risk Fusion

Platt-scaled per-agent anomaly scores.

Bayesian log-odds + context priors.

Temporal smoothing (exponential decay, CUSUM).

🔐 Privacy, Security & Trust

Data minimization: only timings/kinematics, no content.

Secure storage: hardware keystore / secure enclave.

Threats & mitigations:

Replay/bots 🤖 → jitter/periodicity checks.

Shoulder-surf/handover 👀 → motion mismatch → step-up auth.

Poisoning 🧪 → trusted context gating + outlier-robust training.

Model extraction 🛡️ → encryption, attestation, throttle on root/debug.

Consent & compliance: privacy switch + local wipe/reset.

🎛️ UX & Response Policies

Medium Risk: step-up auth 🔑, rate-limit, mask balances/PII.

High Risk: lock ⛔, revoke tokens 🔓, honeypot mode 🎭.

Transparency: explain prompts, privacy dashboard, reset option.

⚡ In short: Sentinel-Edge = 🧠 multi-agent learning + 🔒 on-device privacy + ⚡ real-time detection + 🎯 adaptive responses.

 Implementation Plan & Repo Layout
<img width="1024" height="1536" alt="image" src="https://github.com/user-attachments/assets/62c44196-fc5d-46ec-95ef-a1a76ef2c509" />


 Android (TFLite/LiteRT) Skeleton

Kotlin services per agent via foreground service only when needed; else JobScheduler/WorkManager with battery-aware cadence.

TFLite Interpreter with GPU/NN delegate if available; CPU fallback.

Keystore-backed encryption for feature store; biometric-gated learning.

iOS (Core ML) Skeleton

Swift Combine pipelines to collect events; background tasks for batching.

MLUpdateTask for on-device model updates; ModelConfiguration tuned for low memory.

Feature Engineering

Touch 👆: dwell time, path curvature, velocity quantiles, tremor PSD, pressure deltas, two-finger ratios.
Keystroke ⌨️: KD, UD, DU latencies, digraph/trigraph stats, error bursts, adaptive WPM baseline, backspace/autocorrect cadence.
Motion 📱: energy in 0.8–3 Hz band, device-in-hand vs table heuristics, orientation transitions.
App Usage 📊: session entropy, Markov next-app prob., unusual foreground app timing, notification lag.

 Evaluation Protocol

Datasets (pretrain/benchmark): HMOG (touch+motion), Aalto (keystroke), SHL (locomotion).

User Study: opt-in, with synthetic adversary sessions (scripted taps, emulator, device handover).

Metrics: EER, ROC-AUC per agent; TTD (Time-to-Detect); Battery cost (mWh/hr); False Intervention Rate/day.

Ablations: model vs rule-only; fusion strategies; quantization modes.

 Risk & Edge Cases

Shared device/guest: temp profile, auto-expire.

Cold start: wide thresholds until N trusted sessions; raise friction only for high-risk actions.

Mode changes (injury/new keyboard): warm-up learning rate + volatility detector.

Adaptive Risk-Based Authentication (ARBA)

🎯 Principle → anomaly ≠ auto-block; escalate based on severity.

⚖️ Risk tiers: Low 🟢 (monitor only), Medium 🟡 (step-up auth), High 🔴 (session lock + full re-auth).

🔐 Local-only using BiometricPrompt (Android), LAContext (iOS).

⚙️ Adaptive: context drift, failed unlocks, guest/travel mode.

📊 Metrics: false intervention rate vs detection latency.

🖼️ SentinelEdge App – UI/UX Flow (Technical View)

🚀 Splash Screen / Logo
Background: Dark gradient with glowing 🛡️ shield + 🌊 edge wave icon (symbolizing on-device edge security).

Text: “SentinelEdge – Secure 🔒. Private 🕶️. On-Device 📱.”

Loading Animation: Circular pulse with neural net lines ⚡.

🏠 Home Dashboard

Real-time Risk Status Indicator:

🟢 Safe – No anomalies detected.

🟡 Suspicious – Behavior deviation noted.

🔴 High Risk – Possible fraud attempt.

Quick Summary Panel:

“Behavioral monitoring active ✅ – no anomalies detected.”

Controls:

📊 View Report button.

⚙️ Go to Settings.

⚠️ Anomaly Alerts Screen

Feed of Recent Events:

“📱 Unusual touch pattern detected – 12:45 PM.”

“⌨️ Typing rhythm anomaly – 14:10 PM.”

Severity Badges:

🟢 Low | 🟡 Medium | 🔴 High

Action Buttons:

🛑 Block Activity

🔄 Re-Verify

📑 Detailed Report Screen

Visualization Tools:

📊 Line charts for typing rhythm latency.

📈 Scatter plots for touch pressure vs speed.

📉 Motion activity timeline.

Timeline:

Sequential anomaly logs with timestamps.

Export Options:

⬇️ Save locally

🔒 Encrypted share option.

🔐 Adaptive Risk-Based Authentication (ARBA)

Triggered on 🔴 High Risk event.

Dynamic Verification Modal:

😷 Face ID (Biometric AI verification).

🖐️ Fingerprint scan.

🔢 OTP challenge.

Smart Decision Engine:

Severity-based adaptive flow (e.g., fingerprint for medium anomaly, FaceID + OTP for critical anomaly).

Action Button:

🛡️ Verify Identity.

⚙️ Settings & Customization

Agent Selection:

📱 Touch Pattern Agent.

⌨️ Keystroke Dynamics Agent.

🌀 Motion & Gyro Agent.

📂 App Usage Agent.

Privacy Controls:

“All data remains on-device 🔒 – zero cloud upload.”

UI Preferences:

🌞 Light Mode / 🌙 Dark Mode toggle.

Developer Mode:

Logs, anomaly thresholds tuning ⚡.

Some Snippets of how it will look :
<img width="1024" height="1536" alt="image" src="https://github.com/user-attachments/assets/218a6fbf-0d71-4157-ad48-f506e3728e41" />
<img width="1024" height="1536" alt="image" src="https://github.com/user-attachments/assets/d7076fed-c418-4d29-b0b2-67ac06f9ea74" />
<img width="1024" height="1536" alt="image" src="https://github.com/user-attachments/assets/2c5d2553-396f-4f4a-bc12-3c6f25bbae9f" />
<img width="1024" height="1536" alt="image" src="https://github.com/user-attachments/assets/f1e844e4-01d0-4308-8a5e-82197d51c247" />
In Dark Mode :
<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/bf733fca-210e-4146-aae4-fce58d6fe958" />



