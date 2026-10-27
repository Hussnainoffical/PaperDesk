# PaperDesk design choices and research trail

This MVP applies a small number of practical findings from the local 2026 reading pack. All cited items below are arXiv preprints; this is a design trail, not a claim that the methods are validated for every corpus.

- **Show evidence at page level, and keep answer correctness separate from evidence support.** [The Bottleneck Is Retrieval, Not Reading](https://arxiv.org/abs/2610.26951) argues that answer-only evaluation can hide incomplete evidence and citation failures. PaperDesk stores page references and exposes retrieved passages.
- **Expect a reliability/informativeness trade-off.** [Adaptive Retrieval, Selective Answers](https://arxiv.org/abs/2610.34872) reports that higher support targets reduce answer availability. PaperDesk has an explicit abstention path when the lexical match is too weak; the current threshold is a rough baseline to calibrate, not a guarantee.
- **Make RAG configuration measurable.** [RAG-NIMBF](https://arxiv.org/abs/2610.27816) presents systematic component sweeps and observes that retrieval diversity and prompting can affect quality. This MVP starts with SQLite BM25 and matched terms so a later dense/hybrid/reranking comparison can be measured against the same questions.
- **Treat retrieved content as untrusted.** [nokast-secureRAG](https://arxiv.org/abs/2610.28364) reports strong in-distribution prompt-injection detection but a near-zero result on an external poisoning benchmark. PaperDesk does not claim to defend against prompt injection; any future guard needs out-of-distribution tests.
- **Small models can work on narrow calls, with task-specific trade-offs.** [Not Every Call Needs a Frontier Model](https://arxiv.org/abs/2610.09021) reports call-site-specific results in a deployed agent. PaperDesk makes generation optional and lets the local user choose a compatible model endpoint, while retrieval runs independently.
- **Do not hide tool and harness behavior behind a final score.** [Task Success Is Not Enough](https://arxiv.org/abs/2610.34167) and [When Agents Fail: Model or Harness?](https://arxiv.org/abs/2610.32483) motivate surfacing retrieval evidence and running explicit evaluation rather than presenting a fluent answer as proof.

## MVP choices

1. Local-first SQLite storage and FTS5 keep corpus search usable without an API key.
2. Per-page chunks keep citations simple and inspectable.
3. Optional generation uses an OpenAI-compatible API to support local serving, including Ollama.
4. The UI reports whether it is retrieval-only or model-assisted and warns that valid citation IDs do not prove entailment.
5. Ingestion does not include OCR. This is deliberate: the first version should make searchable text and its provenance reliable before adding multimodal document processing.
