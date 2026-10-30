# Own/fine-tuned model vs frontier APIs for an AI technical interviewer (as of Oct 2026)

Research limits: ~11 tool calls, mostly search snippets (many secondary/vendor blogs); preprints.org fetch failed (host not allowed in session network). Figures marked "Inference" are my own arithmetic, not sourced.

## 1. Are open-weight models (LLMs, speech-to-speech, TTS/STT) good enough for interviewing and evaluating in 2026?

### Takeaway
Open components (Kyutai STT/TTS + Unmute wrapper around an open LLM such as Qwen3/Gemma/Llama) are demonstrably self-hostable today, and an open Qwen3-30B judge matched/beat GPT-4.1 on one interview-coverage judging task. But I found no head-to-head evidence on technical-interview conducting or scoring quality for open vs frontier models, so "good enough" is unproven for this use case.

### Cited Findings
- Kyutai open-sourced Unmute and Kyutai TTS in July 2025; Unmute wraps any text LLM with Kyutai STT/TTS. Kyutai itself notes a text "brain" loses emotion, emphasis, intonation, hesitation, tone; voice cloning not released. — [Kyutai](https://kyutai.org/2025/07/03/tts-unmute-open-source.html), [Kyutai Unmute](https://kyutai.org/unmute/)
- Community self-host example uses Qwen3 via vLLM with Kyutai STT/TTS; a blog describes Gemma 3/Llama 3 via Ollama (secondary sources, not official docs). — [invincible-voice repo](https://github.com/Chareeef/invincible-voice), [Towards AI deep dive](https://pub.towardsai.net/unmute-giving-voice-to-ai-a-deep-dive-into-kyutais-framework-6d8f83bc9531)
- Moshi (7B full-duplex speech-text model): paper reports ~160 ms theoretical / ~200 ms practical latency; Mimi codec 24 kHz audio at 1.1 kbps, 12.5 Hz. PyTorch version reportedly needs ~24 GB VRAM (secondary source). Paper/model license page shows CC BY-NC-SA 4.0 on the arXiv paper (the paper's license, not necessarily the weights'; verify weight licenses before commercial use). — [Moshi paper](https://arxiv.org/html/2410.00037v1), [Spheron summary](https://www.spheron.network/blog/speech-to-speech-gpu-cloud-moshi-sesame-csm-hertz-dev/)
- In SparkMe (LLM semi-structured interviewer, Feb 2026), Qwen3-30B-A3B-Instruct as judge of interview coverage had moderate-to-high agreement with human annotators and beat GPT-4.1 and GPT-4.1-mini; human annotators' Krippendorff alpha was 0.65. — [SparkMe arXiv 2602.21136](https://arxiv.org/pdf/2602.21136)
- Fine-tuned judge models (JudgeLM) reach >90% agreement with a GPT-4 teacher in-domain, but a follow-up finds fine-tuned judges behave like task-specific classifiers and underperform GPT-4 on generalizability, fairness, aspect-specific evaluation. — [JudgeLM](https://arxiv.org/abs/2310.17631), [Huang et al. 2024](https://arxiv.org/abs/2403.02839v2)

### Inferences
- A cascaded pipeline (open STT -> LLM -> open TTS) is the practical open path; true speech-to-speech models (Moshi) are less controllable (hard to inject rubric/state, structured outputs) and conversation intelligence of a 7B model is far below frontier LLMs. Technical interviewing (follow-ups on code/system design) likely needs strong reasoning, favoring frontier or large open models (not verified by evidence found).
- Scoring is the best candidate for open/fine-tuned models (narrow, offline, latency-insensitive); live questioning is the worst.

### Gaps
- No benchmark found comparing Llama/Qwen/DeepSeek/Mistral/Gemma vs frontier on technical-interview questioning or scoring. No verified 2026 model-card comparisons (I did not fetch model cards). Latest Qwen/DeepSeek/Gemma versions and licenses not verified.
- Moshi concurrency claim (64 sessions on L40S) is from one unverified blog; unconfirmed.

## 2. What fine-tuning takes (data, compute, expertise, time) and what it adds over prompting + RAG + rubrics

### Takeaway
I found little hard evidence on fine-tuning cost or data requirements specific to interview scoring. General evidence: fine-tuned/distilled small models match teachers in-domain but generalize worse; rubric-decomposed prompting already gets high agreement with experts, so fine-tuning's marginal gain is mostly cost/latency/privacy, not quality, at the start.

### Cited Findings
- Automated Multiple Mini-Interview scoring: splitting the rubric into single-criterion tasks in a multi-agent prompting setup, with few-shot examples, gave the most accurate results; high agreement with experts but still below human-human levels in prior work. — [arXiv 2602.02360](https://arxiv.org/pdf/2602.02360)
- Teacher-hiring interview scoring: prompting methods reduced scoring variability and aligned strongly with trained human scorers. — [EdWorkingPaper ai26-1385](https://edworkingpapers.com/sites/default/files/ai26-1385.pdf)
- Clinical MADRS interviews: LLM vs trained raters ICC .78 total; item-level agreement 57%-98%; LLM scored higher on average (conference abstract). — [ISCTM abstract](https://isctm.org/public_access/22nd_Annual/Abstract/DeBonis_QC_Abstract.pdf)
- Fine-tuned judges: strong in-domain, weaker transfer (see Q1 sources). OpenAI's built-in distillation only targets OpenAI models; it doesn't export to open models. — [Humanloop](https://humanloop.com/blog/model-distillation), [OpenAI](https://openai.com/index/api-model-distillation/)

### Inferences
- Data sources: (a) own users' sessions (needs explicit consent, retention policy, de-identification of voice/PII); (b) synthetic transcripts from frontier models (cheap, but teaches the teacher's biases); (c) paid expert annotation (the scarce piece: multiple raters per transcript to measure agreement); (d) public datasets are scarce for technical interviews (I did not find one). Labeled scores from experts, not transcripts, are the bottleneck.
- Rough scale (my judgment, unsourced): LoRA on 7-30B models costs hundreds to low thousands of USD of GPU; the real cost is annotation and ML engineer time (weeks to months), plus ongoing re-validation whenever rubric changes.

### Gaps
- No sourced compute-cost, dataset-size, or timeline numbers for fine-tuning interview scorers. No public technical-interview scoring datasets located.

## 3. Self-hosting cost for real-time voice vs API per-minute cost; break-even

### Takeaway
Managed/API voice runs roughly $0.05-0.15/min (more with extras); self-hosted component stacks are claimed at ~$0.02-0.06/min excl. telephony, but those claims come from vendor blogs and exclude engineering/on-call. Hardware alone breaks even at modest volume; the real break-even is driven by engineering headcount.

### Cited Findings
- OpenAI flagship realtime audio (gpt-realtime-2 / 2.1 per secondary sources): $32/M audio in, $64/M audio out, $0.40/M cached in; ~600 audio tokens/min in (~$0.019/min) and ~1,200 out (~$0.077/min); ~$0.05 per conversation minute estimate before context growth. A mini variant reported at $10/$20 per M (one or two sources only). Older sources cite $40/$80. — [Fora Soft](https://www.forasoft.com/blog/article/openai-realtime-api-pricing), [Layer3 Labs](https://www.layer3labs.io/guides/openai-realtime-api-pricing), [Aireiter](https://aireiter.com/blog/openai-realtime-api-pricing)
- Vapi: $0.05/min platform fee plus pass-through (one analysis: advertised $0.05 bills $0.09-$0.14); Retell $0.07+/min, estimator range $0.07-$0.31/min. — [flexprice](https://flexprice.io/pricing-index/vapi), [beri.net](https://www.beri.net/article/voice-ai-per-minute-pricing-vapi-retell-elevenlabs-bland-cost-per-resolved-call), [Retell](https://www.retellai.com/nl/resources/voice-ai-platform-pricing-comparison-2025)
- Vendor (Dograh, sells self-hosted) claims: Deepgram STT ~$0.004-0.006/min, ElevenLabs TTS ~$0.01-0.015/min; Retell-like stack $0.12-0.15/min vs self-hosted $0.03-0.06/min, <$0.02 if models also self-hosted; ~$0.035/min at 100k min/month. Engineering/on-call excluded. — [Dograh](https://blog.dograh.com/self-hosted-voice-agents-vs-retell-real-cost-analysis-tco-tables-min/), [Dograh/Bland](https://blog.dograh.com/self-hosted-voice-agents-vs-bland-real-cost-analysis-100k-minute-tco/)
- GPU rental (Oct 2026 snapshots, secondary aggregators): H100 SXM ~ $2.69 RunPod secure, ~$3.95 Modal (per-second, GPU only), $3.99-4.29 Lambda; marketplaces ~$1.35-2.27/hr with reliability variance. — [Spheron RunPod](https://www.spheron.network/blog/runpod-h100-pricing-2026/), [Spheron Lambda](https://www.spheron.network/blog/lambda-cloud-h100-pricing-2026/), [IntuitionLabs](https://intuitionlabs.ai/articles/h100-rental-prices-cloud-comparison), [computeprices](https://computeprices.com/providers/runpod/gpus/h100)
- Moshi on one L4 ~200 ms (secondary); 64 concurrent on L40S (unverified blog). — [Spheron](https://www.spheron.network/blog/speech-to-speech-gpu-cloud-moshi-sesame-csm-hertz-dev/)

### Inferences (my arithmetic; assumptions flagged)
- An always-on H100 at ~$3/hr is ~$2,200/month. Assume (not measured) 20 concurrent cascaded sessions at 25% average utilization: 20 x 0.25 x 730 h = ~3,650 session-hours/month, i.e. ~$0.01/session-minute. Break-even vs a $0.08/min API is then ~27k minutes/month (~900 thirty-minute interviews) on hardware alone. Adding one ML/infra engineer (~$15-20k/month loaded, assumption) pushes true break-even to roughly 200k+ minutes/month (~7k+ thirty-minute interviews). Utilization (spiky demand) and per-GPU concurrency are the swing variables.
- Mock-interview sessions are long (30-60 min), so per-session API cost of ~$3-8 is meaningful against consumer pricing; this pressures margin earlier than short-call use cases. Mitigate first by cascading cheaper STT/TTS + small LLM via API, before self-hosting.

### Gaps
- No measured concurrency-per-GPU for a full open cascade (Kyutai STT + 8-30B LLM + Kyutai TTS). Official OpenAI pricing page was not fetched (figures from third-party blogs).

## 4. Evaluation quality: reliability vs human interviewers, bias/fairness, hallucination control

### Takeaway
LLM scoring agrees "moderately to well" with humans on average but varies by item and has documented reliability failures; bias evidence is mixed and model-version dependent; legal audit obligations already exist for hiring uses (less so for pure practice tools).

### Cited Findings
- July 2026 preprint auditing LLM behavioral-interview scoring reports five reliability failures including scale collapse, test-retest inconsistency, cross-model disagreement (snippet only; could not fetch full text). — [Preprints 202607.1567](https://www.preprints.org/manuscript/202607.1567)
- RACER: 72% agreement with two human evaluators vs 77% human-human. — [arXiv 2402.02656](https://arxiv.org/pdf/2402.02656)
- Human reference reliability can be modest (alpha 0.65 for annotators in SparkMe). — [SparkMe](https://arxiv.org/pdf/2602.21136)
- Bias: UW 2024 study (press-reported) found open-source LLMs favored white-associated names 85% and female-associated names 11% of the time in resume ranking; a June 2026 14-model paired-resume audit finds race and gender gaps correlated across models and bias direction flips across model generations; a July 2026 preprint finds very small bias in commercial and one open-weight model, with an open-weight model reacting to explicit minority-status signals. Results conflict by method. — [GeekWire](https://www.geekwire.com/2024/ai-overwhelmingly-prefers-white-and-male-job-candidates-in-new-test-of-resume-screening-bias/), [arXiv 2606.28978](https://arxiv.org/pdf/2606.28978), [Preprints 202607.0694](https://www.preprints.org/manuscript/202607.0694)
- Regulation: NYC Local Law 144 (in force) requires annual independent bias audit and candidate notice for automated employment decision tools; EU AI Act classes recruitment/candidate evaluation as high-risk, with obligations reportedly delayed (one source: Dec 2027) pending final adoption. — [Sonomos](https://sonomos.ai/blog/ai-hiring-eeoc-nyc-ll144-eu-ai-act-2026/), [Morgan Lewis](https://www.morganlewis.com/pubs/2026/06/changes-to-eu-ai-act-deadlines-what-it-means-for-employers-and-hr-technology-providers)

### Inferences
- Validation plan (standard psychometrics, not from a specific source): double-human-score a held-out set (>=2 raters), report ICC/weighted kappa/QWK of model vs each human and human-human; test-retest across repeated runs and temperature; paraphrase and name/accent perturbation tests; disaggregated error by group; keep scoring tied to evidence quotes (structured output requiring transcript spans) to curb hallucinated feedback; re-validate on every model/prompt change.
- Fine-tuning a scorer on one rater pool can bake in that pool's bias; fine-tuned judges' generalization issues (above) mean per-role/rubric revalidation.
- Positioning as a practice/coaching tool (not employer screening) likely lowers legal exposure; that is a legal-judgment point, verify with counsel.

### Gaps
- Full text of the 2026 interview-scoring reliability audit not read. No head-to-head open vs frontier reliability data for interview scoring. Accent/ASR-bias effects on STT-fed scoring not researched.

## 5. Hybrid strategy and startup examples

### Takeaway
The "API first, log data, distill narrow components later" pattern is widely described and tooled (OpenPipe), with named adopters, but I found no verified case study from an interview/edtech startup and the 90% savings claim is unverified.

### Cited Findings
- Sacra interview with OpenPipe CEO: Ramp, Notion, Databricks reportedly used frontier outputs to fine-tune open models (Mistral, Llama 3.1) for specific tasks, cutting costs ~90% (snippet truncated; interview claim, not a published case study). — [Sacra](https://sacra.com/research/kyle-corbitt-openpipe-fine-tuning-llms/)
- Vendor distillation features exist (OpenAI) but are locked to the vendor's own small models. — [OpenAI](https://openai.com/index/api-model-distillation/), [Humanloop](https://humanloop.com/blog/model-distillation)

### Inferences
- Sensible sequence: (1) ship on APIs with rubric-decomposed structured scoring; (2) log transcripts, model outputs, and any human corrections with consent; (3) build a human-labeled gold set early (needed for any approach); (4) distill scoring and question-selection/classification first (offline, narrow), then cheaper STT/TTS; (5) keep a frontier model for live conversational turns and as fallback/audit.

### Gaps
- No named interview/mock-interview startup found that publicly describes this migration. Not researched: competitor architectures (e.g., Final Round, Interviewing.io, HireVue, Micro1) and whether they use custom models.

## 6. Data moat and defensibility

### Takeaway
I found no direct evidence on moats for AI interview products. Reasoning only: raw transcripts are a weak moat; expert-labeled outcomes (did candidates get offers, calibrated against real interviewer scores) and distribution/trust are stronger.

### Cited Findings
- Fine-tuned judges are task/rubric-specific and generalize poorly (so a custom scorer's advantage is narrow and must be maintained). — [Huang et al. 2024](https://arxiv.org/abs/2403.02839v2)
- Frontier outputs can be used to cheaply train open student models, which lowers barriers for any competitor who can generate synthetic data. — [Sacra](https://sacra.com/research/kyle-corbitt-openpipe-fine-tuning-llms/)

### Inferences (speculation)
- Candidate moats: longitudinal per-user performance data; outcome labels (interview results, job offers) linked to practice behavior; company-specific question banks and calibrated rubrics from expert interviewers; brand/community; B2B integrations. Weak moats: prompts, generic transcripts, voice UI.
- Model quality advantage is likely temporary as frontier APIs improve; consent restrictions and privacy law limit how much user data can be reused for training.

### Gaps
- No empirical study or startup disclosure on data-moat effects in this category.
