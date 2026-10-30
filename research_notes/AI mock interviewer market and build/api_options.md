# Third-party API options for a real-time voice AI mock interviewer (notes dated 2026-10-30)

**Research limits (read first):** Direct fetching of vendor sites (openai.com, vapi.ai, livekit.io, retellai.com, elevenlabs.io, deepgram.com, cartesia.ai, ai.google.dev) failed in this session: those hosts are outside the session's allowed network domains. All figures below come from WebSearch result summaries of third-party blogs and a few OpenAI/Google doc pages that appeared in results but were not opened. None were verified against official pricing pages. Third-party sources frequently disagree. Treat every number as "reported, unverified" unless stated; all cost-per-interview and margin numbers are MY ESTIMATES built on those inputs. To firm up, allow those hosts and re-run.

## 1. Architectures: speech-to-speech (S2S) vs cascaded STT -> LLM -> TTS

### Takeaway
S2S is faster on paper (reported 300-700 ms vs 600-1,200 ms typical for cascaded; some sources say 1.5-3 s for real production cascades) but cascaded wins on controllability, transcripts, LLM choice and, usually, predictable cost. For an interviewer that needs rubric-driven questioning and a reliable transcript for later scoring, cascaded (or S2S with a parallel transcription stream) is the safer default.

### Cited Findings
- Typical end-to-end latency reported as 600-1,200 ms for cascaded and 300-700 ms for S2S — [Coval](https://coval.ai/blog/voice-ai-models-2026/); a different source says practical production cascades are 1.5-3 s from end of user speech to start of agent audio — [Gradium](https://gradium.ai/content/cascaded-voice-agent-vs-speech-to-speech-2026)
- Twilio (Nov 2025) cascaded component targets: TTS TTFB 100 ms, LLM time-to-first-token 375 ms, STT 350 ms, mouth-to-ear turn gap target 1,115 ms — [Twilio](https://www.twilio.com/en-us/blog/developers/best-practices/guide-core-latency-ai-voice-agents)
- Endpointing, VAD and STT finalisation add delay beyond model inference; ElevenLabs TTS TTFB measured ~220 ms independently — [SoftwareSeni](https://www.softwareseni.com/how-realtime-voice-agents-work-and-why-latency-drives-vendor-choice/)
- On real phone lines the S2S latency gap shrinks because other delays dominate; a March 2026 benchmark (single source, verify) found S2S agents completed 26-38% of tasks under noise/accents vs 85% for a text-based agent — [JustCall](https://justcall.io/blog/speech-to-speech-vs-cascaded-voice-agents.html)
- Named S2S models: OpenAI gpt-realtime family, Google Gemini Live, Amazon Nova 2 Sonic, xAI Grok voice — [JustCall](https://justcall.io/blog/speech-to-speech-vs-cascaded-voice-agents.html)
- Cascade is the enterprise default because it keeps transcripts and lets teams swap LLMs — [JustCall](https://justcall.io/blog/speech-to-speech-vs-cascaded-voice-agents.html); a paper reports cascaded ~17x faster to first audio than Qwen2.5-Omni-7B and notes native models still lag on tool use — [EmergentMind](https://www.emergentmind.com/papers/2603.05413)
- OpenAI Realtime pricing (reported, unverified): gpt-realtime-2.1 $32/M audio input, $64/M audio output, cached input $0.40/M; gpt-realtime-2.1-mini $10/$20 per M. Release dates conflict (July 6 vs Sept 26, 2026). Per-minute conversions are third-party estimates (~$0.019/min heard, ~$0.077/min spoken) — aggregated at [Forasoft](https://www.forasoft.com/article/openai-realtime-api-pricing), [Layer3Labs](https://www.layer3labs.io/guides/openai-realtime-api-pricing), [OpenAI model docs](https://developers.openai.com/api/docs/models/gpt-realtime) (not opened)
- Gemini Live (reported): audio ~25 tokens/s; $3.00/M audio input (~$0.005/min), $12.00/M audio output (~$0.018/min); model naming (2.5 Flash Native Audio vs "3.1 Flash Live Preview" vs "3.8 Live") conflicts across sources; silence/session time may be billable; one forum thread asks whether cumulative prompt tokens are re-billed each turn — [RyWalker](https://rywalker.com/research/gemini-live-api), [TokenKarma](https://tokenkarma.app/blog/gemini-live-api-pricing-voice-agents-2026/), [Google forum](https://discuss.ai.google.dev/t/does-gemini-live-native-audio-bill-cumulative-prompt-tokens-on-every-turn-cost-seems-to-scale-with-turn-count-not-call-duration/173248)
- Barge-in/interruption: no vendor-neutral benchmark found. Orchestration frameworks (LiveKit, Pipecat, Vapi, Retell) expose VAD/turn-detection tuning; S2S APIs have server-side VAD. Not verified in this session.

### Inferences
- A 45-minute session is long-context: S2S models that re-read accumulated audio context each turn can cost far more than the naive per-minute math (cached input at $0.40/M mitigates for OpenAI). Hidden-cost risk is higher for S2S than cascaded.
- Interview use has long candidate silences while thinking/coding; tune endpointing so the agent does not interrupt, and check whether silence is billed (Gemini: possibly).
- Hybrid is plausible: cascaded for the interviewer turn loop, with a stronger LLM (Claude/GPT/Gemini Pro) run asynchronously on the transcript for evaluation.

### Gaps
- Official, verified pricing and independent head-to-head barge-in quality benchmarks not obtained (sites blocked).
- Transcript quality of S2S (its input transcription is a separate Whisper/gpt-transcribe pass) not verified.

## 2. Orchestration platforms

### Takeaway
Reported all-in cost of managed platforms is ~$0.07-0.31/min versus ~$0.01/min platform fee on LiveKit Cloud (open-source Agents framework), but LiveKit/Pipecat require engineering. Vapi's $0.05/min headline excludes STT/LLM/TTS/telephony.

### Cited Findings
- Vapi: $0.05/min platform fee excludes STT, LLM, TTS, telephony; realistic all-in ~$0.09-0.14/min (one source $0.05-0.12 runtime + models) — [Beri](https://www.beri.net/article/voice-ai-per-minute-pricing-vapi-retell-elevenlabs-bland-cost-per-resolved-call), [Morph](https://www.morphllm.com/best-ai-voice-agent-platforms)
- Retell: itemized; ~$0.055/min platform line, advertised range $0.07-0.31/min depending on voice/LLM/telephony — [Techsy](https://techsy.io/en/blog/ai-voice-agent-pricing), [Morph](https://www.morphllm.com/best-ai-voice-agent-platforms)
- ElevenLabs Agents: ~$0.08/min hosting listed by one source; bundled platforms $0.10-0.18/min; LLM often extra — [Morph](https://www.morphllm.com/best-ai-voice-agent-platforms), [Techsy](https://techsy.io/en/blog/ai-voice-agent-pricing)
- LiveKit Cloud: starts at $0, about $0.01 per agent-session minute after allowances; Agents framework open source (lowest lock-in, most engineering) — [Ecorpit](https://ecorpit.com/ai-voice-agent-platforms-vapi-retell-bland-livekit-2026/), [Speechify](https://speechify.ai/blog/best-voice-agent-platforms-2026)
- Twilio US outbound telephony ~$0.014/min (not needed for a web/app-based interviewer using WebRTC) — [Techsy](https://techsy.io/en/blog/ai-voice-agent-pricing)
- Deepgram bundled Voice Agent API listed at $0.075/min — [HappyRobot](https://www.happyrobot.ai/hub/deepgram-pricing)

### Inferences
- Vapi, Retell, Bland are phone-call oriented (telephony, campaigns); a browser-based interviewer with code editor/whiteboard is better served by LiveKit Agents or Pipecat (WebRTC, data channels, open source, low lock-in). Daily (Pipecat's sponsor) not priced here.
- Lock-in ranking (my judgment): Pipecat/LiveKit self-host lowest; LiveKit Cloud low; Vapi/Retell/ElevenLabs Agents higher.

### Gaps
- Daily, Bland pricing and verified feature lists not found; no official pages read.

## 3. Component vendors

### Takeaway
STT is a small cost line (~$0.005-0.008/min); TTS (~$0.03-0.05/min) and LLM dominate cascaded cost.

### Cited Findings
- Deepgram Nova-3 streaming: $0.0048/min (monolingual/promo) to $0.0077/min standard (July 2026); Aura-2 TTS $0.030 per 1K chars — [Speechify alternatives](https://speechify.ai/alternatives/deepgram), [Costbench](https://costbench.com/compare/cartesia-vs-deepgram/), [Fast.io](https://fast.io/resources/best-ai-text-to-speech-tools-2026/)
- ElevenLabs Flash/Turbo ~$50 per 1M chars (~$0.045/min assuming ~900 chars/min), ~75 ms claimed — [Fast.io](https://www.fast.io/resources/best-ai-text-to-speech-tools-2026.md)
- Cartesia Sonic ~$39/M chars on Startup plan (~$0.035/min); claims 40-90 ms TTFA, but one independent benchmark reported P50 ~188 ms — [Fast.io](https://fast.io/resources/best-ai-text-to-speech-tools-2026/), [Techsy IT](https://techsy.io/it/blog/migliori-api-sintesi-vocale-sviluppatori)
- Sources conflict on whether Deepgram or Cartesia is cheaper for TTS — [Costbench](https://costbench.com/compare/cartesia-vs-deepgram/)

### Inferences
- Reasonable LLM pricing was NOT sourced this session. From my background knowledge only (unverified, pre-2026): small/fast models (GPT-mini class, Gemini Flash, Claude Haiku) cost roughly $0.1-1 per M input tokens and $0.4-5 per M output; frontier models (Claude Sonnet/Opus, GPT flagship, Gemini Pro) roughly $2-15 input, $10-75 output per M. Verify before use. Use a fast model for live turns (TTFT matters) and a stronger model once post-session.
- Whisper/AssemblyAI/Rime/OpenAI TTS pricing not obtained.

### Gaps
- LLM pricing/quality for Claude, GPT, Gemini in 2026; AssemblyAI, Rime, OpenAI TTS pricing; independent STT WER benchmarks (accents, technical vocabulary).

## 4. Estimated cost per 45-minute mock interview (ALL ESTIMATES)

Assumptions: interviewer speaks ~40% of time (18 min, ~16k chars), candidate ~50%; STT billed for entire session; context grows over the session.

| Architecture | Components | Est. cost / 45 min |
|---|---|---|
| S2S flagship (gpt-realtime-2.1) | naive audio math: in ~$0.9 + out ~$1.4 = ~$2.3; with context re-reads/tool calls | ~$3-8 (high uncertainty) |
| S2S mini (gpt-realtime-2.1-mini, ~1/3 price) | | ~$1-3 |
| Gemini Live | in ~$0.23 + out ~$0.3 naive (~$0.5); reasoning/tools/context re-billing may multiply | ~$1-4 (high uncertainty) |
| Cascaded DIY on LiveKit/Pipecat | STT ~$0.35 + TTS ~$0.5-0.8 + LLM ~$0.3-1.5 + LiveKit ~$0.45 | ~$1.6-3.1 |
| Cascaded on Vapi | adds $0.05/min = $2.25 | ~$3.5-6 |
| Post-interview evaluation (frontier LLM, ~15k token transcript + rubric) | | ~$0.05-0.40 |
| Code sandbox, storage, misc. | | ~$0.05-0.20 |

### Gross margin at price points (my arithmetic; assumes cost midpoints: cascaded DIY ~$2.5, S2S mini ~$2, S2S flagship ~$5, Vapi ~$4.75)
| Price | Sessions | Cascaded DIY | S2S mini | S2S flagship | Vapi |
|---|---|---|---|---|---|
| $10/mo | 2 | 50% | 60% | 0% | 5% |
| $20/mo | 4 | 50% | 60% | 0% | 5% |
| $40/mo | 8 | 50% | 60% | 0% | 5% |
| $15 per session | 1 | 83% | 87% | 67% | 68% |
Note: per-subscription margin falls as the number of included sessions rises; a monthly plan should cap sessions/minutes (e.g. 4 x 45 min on $20 gives ~50% at DIY cascaded; 3 sessions would give ~62%). Add 3-5% payment fees and infra.

### Gaps
- All inputs unverified; real usage data (tokens/min, silence share) needed. Run a pilot of 10 sessions to measure.

## 5. Extras (code editor, whiteboard, video, scoring)

### Takeaway
Not researched with sources in this session (budget used on blocked fetches). Only design suggestions below.

### Inferences (no citations; from general knowledge, unverified)
- Code editor: Monaco/CodeMirror in-browser; execution via a hosted sandbox API (e.g. Judge0, E2B, Piston) or WASM/Pyodide; stream editor state to the agent via LiveKit data channel / Pipecat messages so the interviewer can comment on code.
- Whiteboard: tldraw or Excalidraw; export snapshots/JSON to a vision-capable LLM for system-design feedback.
- Video/proctoring: LiveKit/Daily video tracks; optional recording; privacy/consent (BIPA/GDPR) issues for any face analytics.
- Scoring: structured-output LLM call (JSON schema) per rubric dimension with transcript-cited evidence; run async after session; calibrate against human-scored samples.

### Gaps
- No vendor docs, pricing or comparisons gathered for sandboxes, whiteboards, or proctoring.
