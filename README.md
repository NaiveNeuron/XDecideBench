# XDecideBench

[![arXiv](https://img.shields.io/badge/arXiv-2611.01370-b31b1b.svg)](https://arxiv.org/abs/2611.01370)
[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](LICENSE)

**XDecideBench** is a multilingual benchmark for Typed Decision Models: models that return a probability for each answer an application declares, such as one of *k* intents, yes or no, or a level on an ordinal rubric. It has nine suites in 122 languages, with 88,425 texts and 153,875 typed questions, and scores the returned probabilities with chance-normalised Brier skill. We use it to ask three questions. Do model rankings hold across tasks and languages? How much do open decision models gain over the Qwen backbones they were fine-tuned from? Does a confidence threshold fitted on English data keep its error rate in other languages? The paper evaluates eleven Typed Decision Models, including Jev, Clef, pplx-decider and Liquid d1, together with backbone readouts and supervised comparators.

- **Paper:** [arXiv:2611.01370](https://arxiv.org/abs/2611.01370) ([PDF](https://arxiv.org/pdf/2611.01370))
- **Authors:** Marek Šuppa (Comenius University in Bratislava, Cisco Systems) and Jaroslav Kopčan (Kempelen Institute of Intelligent Technologies)
- **Code, manifests and scorer:** not public yet. This repository will hold them. Until then it holds the paper's summary and citation.

## Key findings

All numbers are from the paper. Brier skill is relative to chance (or to a base-rate predictor for skewed yes/no and score suites), so 0 means no better than the reference.

- **Rankings change from suite to suite.** Kendall τ between the Core12 ranking and the rankings on the other suites is only .33 (RTP-LX) to .64 (NLI, Injongo). GPT-6 Luna is eighth on Core12 but first on the native INCLUDE exams.
- **Choosing a model on English is risky.** If we pick the best English model on each parallel suite, another model beats it significantly in 56 of 232 suite–language cells. On Belebele, Microsoft's Decision-1 is best in English but loses significantly in 37 of 105 languages. In Burmese it reaches Brier skill .336, against .814 for Jev.
- **Open decision heads beat their own backbones.** Clef, Clef-flash and pplx-decider beat their Qwen backbones in all twelve Core12 languages, by 7.6 to 18.2 accuracy points under the local readout. A pre-registered held-out cohort confirms that the accuracy gain is larger in the four languages with the least web text than in the six with the most. In both groups, however, the heads fix a similar share of the backbone's errors, so the larger gain may reflect weaker backbones.
- **English-fitted confidence thresholds do not transfer.** A threshold set on English data for 5% errors admits 7.8–12.4% errors on German and French XNLI, and more in low-web-share languages. On MASSIVE, error rates stay near target, but the models admit fewer decisions. Jev admits 10.8% of Swahili decisions against 46.8% of English ones.

## Benchmark at a glance

| Suite | Label regime | Languages | Texts | Typed questions |
|---|---|---:|---:|---:|
| Core12 (MASSIVE, XNLI, Belebele, MMLU-ProX, SIB-200) | Aligned | 12 | 26,460 | 26,460 |
| Wide-Belebele | Aligned | 106 | 10,600 | 10,600 |
| MASSIVE | Aligned | 52 | 6,084 | 6,084 |
| NLI (XNLI, AfriXNLI, AmericasNLI) | Aligned | 37 | 15,222 | 15,222 |
| RTP-LX toxicity | Locally labelled | 38 | 7,600 | 60,800 |
| INCLUDE | Native | 44 | 21,079 | 21,079 |
| InjongoIntent | Native | 16 | 1,920 | 1,920 |
| Multilingual HateCheck + Aya red-teaming | Native | 17 | 2,960 | 2,960 |
| BRIGHTER emotion intensity | Native | 10 | 2,500 | 14,750 |
| **All headline suites** | | **122** | **88,425** | **153,875** |

The NLI suite reuses the 6,000 Core12 XNLI texts, so the distinct totals are lower than the column sums. A separate held-out cohort of 4,860 cases in the Core12 languages was frozen before any inference and run only after pre-registration.

## Method

- **Typed decisions.** Each request declares its answers: one of *k* named options (choice), yes or no, or a level on an ordered rubric (score). The model returns a probability for each, and only those probabilities are scored.
- **One scoring contract for all suites.** A suite is scored only when a submission covers every case. Refusals, interface limits and invalid outputs count as a uniform distribution, and the share of valid outputs is reported beside each score.
- **Metric.** Brier skill score, `BSS = 1 − BS(model) / BS(reference)`, with uniform chance as the reference for choice decisions and the cell's base rate for skewed yes/no and score decisions. Scores are averaged with equal weights from items up to suites, never across suites.
- **Uncertainty.** Every mean has a 95% cluster-bootstrap interval and a signature that fixes how it was computed.
- **Three label regimes, reported separately.** Aligned suites use the same items with one label in every language. RTP-LX aligns items but uses labels from local annotators. Native suites contain items written in each language for its speakers, so their raw scores are never compared across languages.
- **Confidence policies.** A risk threshold for 5% errors and a split conformal set are fitted on English data and tested in other languages.

## Evaluated models

| Model | Access in the paper | Backbone | Source |
|---|---|---|---|
| Clef | local and API | Qwen3.8-27B (locally verified) | [Cloudflare](https://developers.cloudflare.com/changelog/post/2026-10-01-clef-workers-ai/) |
| Clef-flash | local and API | Qwen3.5-9B (locally verified) | [Cloudflare](https://developers.cloudflare.com/changelog/post/2026-10-01-clef-workers-ai/) |
| pplx-decider v1.1 | API | Qwen3.8-27B (stated by vendor) | [OpenRouter](https://openrouter.ai/perplexity/pplx-decider-v1.1-27b) |
| Microsoft Decision-1 | API | Qwen3.5-9B (stated by vendor) | [Microsoft](https://commandline.microsoft.com/microsoft-decision-1-model-foundry/) |
| Kev-4B | API | Qwen3.5-4B (stated by vendor) | [GitHub](https://github.com/jaredpalmer/kev) |
| Laya-multilingual | local | mmBERT-base | [Hugging Face](https://huggingface.co/convaiinnovations/laya) |
| Jev 1.13 | API | undisclosed | [TypeSafe](https://typesafe.ai/blog/introducing-system-one-models-and-jev) |
| GPT-6 Luna Decisions | API | undisclosed | [OpenAI](https://developers.openai.com/api/docs/guides/decisions) |
| Liquid d1 | API | undisclosed | [OpenRouter](https://openrouter.ai/liquid/d1) |
| Mercury Decide | API | undisclosed | [OpenRouter](https://openrouter.ai/inception/mercury-decide) |
| Solar Decide | API | undisclosed | [OpenRouter](https://openrouter.ai/upstage/solar-decide) |

The paper also scores the Qwen3.8-27B and Qwen3.5-9B backbones (local and hosted readouts), a supervised mmBERT comparator and two MediaPipe Decision Maker arms. API models are dated snapshots, which vendors may retire.

## Results

Brier skill per suite, from the main results table of the paper. Higher is better and 0 is the reference. Scores are not comparable across columns because the suites differ in reference and population. See the paper for 95% intervals, valid-output coverage and footnotes.

| Model | Core12 | Belebele | MASSIVE | NLI | RTP-LX | INCLUDE | Injongo | Moder. | BRIGHTER |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| pplx-decider | **.712** | .756 | .718 | **.397** | .051 | .536 | .578 | **.673** | .144 |
| Jev | .688 | **.785** | .619 | .128 | −.101 | .531 | .499 | .561 | −.428 |
| Clef | .674 | .701 | .714 | .254 | .045 | .459 | .492 | .274 | **.167** |
| d1 | .663 | .711 | .634 | .315 | .070 | .488 | .481 | .449 | .118 |
| Decision-1 | .651 | .716 | .721 | .108 | −.229 | .400 | .417 | .500 | −.375 |
| Clef-flash | .631 | .597 | **.745** | .127 | **.071** | .385 | .415 | .379 | .123 |
| Mercury | .611 | .677 | .651 | .188 | −.022 | .411 | .416 | .449 | .013 |
| Luna | .568 | .743 | .602 | −.058 | −.018 | **.600** | **.580** | .541 | −.096 |
| Kev-4B | .449 | .389 | .371 | .070 | −.169 | .273 | .225 | .278 | −.275 |
| Solar | .398 | .334 | capped | −.236 | −.568 | .251 | capped | −.046 | −1.396 |
| Laya | .119 | −.167 | −.120 | −.206 | −1.647 | −.085 | −.096 | −.415 | −2.907 |
| Qwen3.8-27B (hosted readout) | .584 | .678 | .523 | .043 | −.336 | .426 | .405 | .397 | −.056 |
| Qwen3.5-9B (hosted readout) | .477 | .525 | .313 | −.036 | −.562 | .310 | .167 | .347 | −.034 |

Bold marks the best hosted Typed Decision Model per suite. "Capped" means every item exceeds Solar's option limit.

## Language coverage and low-resource limits

The headline suites cover 122 languages. Wide-Belebele (106 languages), the NLI family (37, including AfriXNLI and AmericasNLI) and InjongoIntent (16 African languages) reach the languages with the least web text. The pre-registered head–backbone analysis groups the twelve Core12 languages by web share. Its low-web-share group has four languages, and only Swahili lies below tier 3 in the Joshi et al. (2020) taxonomy. Most aligned suites are translated or localized from English, and Core12 requests pair target-language content with English instructions and options, so language effects combine language with translation artefacts.

## How XDecideBench differs from other decision-model benchmarks

- **Earlier studies cover one model across languages, or several models in English.** Deußer et al. evaluate Jev alone on 37 datasets in many languages. JevBench, S1MB and Decision Index compare several models in English. system-one-benchmark compares five models in eight languages, on choice decisions only.
- **XDecideBench runs eleven models on the same multilingual items**, for all three decision types, and reports aligned and native evidence separately.
- **It pairs open heads with their own backbones** and confirms the larger accuracy gain in low-web-share languages on a pre-registered held-out cohort.
- **It tests whether English-fitted confidence policies transfer across languages**, not only across tasks.

## Reproducibility and artifacts

The manifests, the scorer, the model adapters and the run receipts are not public yet. They will be released in this repository. The manifests hold case identifiers and hashes, not benchmark text, so the cases are rebuilt from the pinned upstream sources, each under its own licence. This section will link the evaluation guide once the code is public.

## Citation

```bibtex
@misc{suppa2026xdecidebench,
  title         = {{XDecideBench}: Evaluating Typed Decision Models and Confidence Policies Across Languages},
  author        = {{\v{S}}uppa, Marek and Kop{\v{c}}an, Jaroslav},
  year          = {2026},
  eprint        = {2611.01370},
  archivePrefix = {arXiv},
  primaryClass  = {cs.CL},
  url           = {https://arxiv.org/abs/2611.01370}
}
```

## FAQ

**Is there a benchmark for Typed Decision Models such as Jev in languages other than English?**
Yes. XDecideBench scores Jev and ten other Typed Decision Models on the same items in 122 languages. Jev is second on Core12 and best on Wide-Belebele, but below the base rate on RTP-LX and BRIGHTER.

**Which decision model is best for multilingual classification or routing?**
No single model is best everywhere. pplx-decider leads Core12, Jev leads Wide-Belebele, Clef-flash leads MASSIVE intent routing and GPT-6 Luna leads the native INCLUDE exams. The paper recommends choosing on suites that match your decision type and setting.

**Can I choose a model on English results and deploy it in other languages?**
Not safely. The best model in English loses significantly in about a quarter of suite–language cells.

**Can I fit a confidence threshold on English data and reuse it in other languages?**
Not without checking. A threshold set for 5% errors on English admits 7.8–12.4% errors on German and French XNLI, and more in languages with less web text. On MASSIVE, it keeps the error rate but admits fewer decisions.

**How does XDecideBench handle low-resource languages?**
Wide-Belebele, the NLI family and InjongoIntent reach languages with very little web text. Native suites are reported separately from translated ones, and raw native scores are not compared across languages.

**Do fine-tuned decision models beat the LLMs they were built from?**
In this panel, yes. Clef, Clef-flash and pplx-decider beat their Qwen backbones in all twelve Core12 languages, with larger accuracy gains in low-web-share languages but a similar share of errors fixed.

**Which metric does XDecideBench use?**
Chance-normalised Brier skill over the full probability vector, with failures scored as uniform and valid-output coverage reported beside each score.

**Can I score my own model?**
Not yet. The scorer and manifests will be released in this repository.

## License

Apache License 2.0 for the contents of this repository. Benchmark items come from upstream datasets under their own licences.
