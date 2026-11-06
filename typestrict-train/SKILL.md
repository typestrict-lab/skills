---
name: typestrict-train
description: Train, follow and download a typestrict decision model with the typestrict CLI or MCP server. Use when the user asks to train a model on their data with typestrict, to check how a training is going, or to get its result.
---

# Train a typestrict model

Use the typestrict MCP tools when they are available; otherwise the CLI, with `--json` when you need to read the
answer. Each step below gives both. The MCP server is remote (`https://api.typestrict.ai/mcp`, the user signs in with
OAuth) or local (`typestrict mcp`, with the CLI's key); setup for each assistant: https://docs.typestrict.ai/agents.

```bash
command -v typestrict || curl -fsSL https://typestrict.ai/install.sh | sh
typestrict credits            # checks the key; if refused, ask the user to run `typestrict login`
```

Never print, write or commit the user's API key.

## 1. Send the data

Prepare it first (skill `typestrict-prepare-data`), then:

```bash
typestrict train data/tickets.csv --goal "Which team handles the ticket?" --watch
# public sources: --hf org/name:data/train.csv   --github org/name
# several decisions: --question "..." --question "..."
```

**Several decisions, one model.** Up to 12 questions about the same text train as one model, answered in one call. It
reads up to 4,096 tokens of each example and is compared with Jev decision by decision: it beats Jev when it beats it on
at least two thirds of them (4 of 5, 2 of 3, both of 2). Prefer it when the decisions are made on the same input.

MCP: `create_dataset`, then `wait_for_change`. Keep the dataset id. With the local server pass file paths in `files`;
with the remote one, pass the prepared table's text in `contents` (`name` with its extension, `text`). Files the remote
server cannot take that way (Excel, PDF, zip, very large ones): ask the user to upload them at
https://typestrict.ai/trainings/new and continue from `list_trainings`. Public sources: `hf`, `github`.

## 2. Follow next_action

`typestrict status <id> --json` (MCP: `get_training`) returns `step` and `next_action`. Act on it:

| `next_action` | CLI | MCP |
| --- | --- | --- |
| `wait` | `typestrict status <id> --watch` | `wait_for_change` |
| `structure` | `typestrict structure <id>`, or `typestrict suggest <id>` to see decisions first | `structure_dataset`, or `suggest_decisions` |
| `pick_decisions` | `typestrict review <id>`, then `typestrict structure <id> --question "..."` | `get_training` (`structure.suggestions`), then `structure_dataset` with `questions` |
| `label_examples` | The user runs `typestrict label <id>` (interactive) | Ask the user each example: `next_example_to_label`, `label_example` |
| `start_training` | Step 3 | Step 3 |
| `start_over` | The files could not be read: show the error and send fixed files | Same |
| `expired` | The data expired (90 days without use): send it again | Same |

Before training, show the user the agent's proposal (`typestrict review <id>`, MCP `get_training`: `structure.spec` and
`structure.stats`): the question, the answers and how many examples each has, and the warnings. To change it:
`typestrict structure <id> --feedback "..."` (MCP `structure_dataset` with `feedback`).

### Make it a strong model before training

Training takes a credit and some hours. As soon as the proposal is ready (before the user labels anything), go through
it yourself and fix what you find with the user (`typestrict structure <id> --feedback "..."`, MCP `structure_dataset`
with `feedback`), then read it again. Each round reruns the agent: gather everything you want changed and send it in one
feedback.

1. **Examples per answer** (`structure.stats.per_label`). Hundreds per answer is where the model gets good, and an
   answer needs at least 5 test examples to be measured. The test examples are 20% of the data (up to 2,000) when it
   came with answers, or a random 150 the user labels when it did not: then an answer under about 3% of the data gets
   fewer than 5. Merge a rare answer with its closest one, or ask the user for more examples of it.
2. **Answers that overlap.** Read the samples (`structure.stats.samples`, 3 per answer). If two answers have samples a
   person could swap, merge them or rewrite their descriptions until the line between them is clear. When the data had
   no answer, `prelabel_agree` against `examples` shows how often the pre-labelers agreed: low agreement means the
   answers or their descriptions are ambiguous.
3. **Labels that look wrong.** A sample whose label does not match its text means the answer column is noisy or the
   question is off: say so and fix the source of the answer.
4. **Every warning** (`structure.stats.warnings`): each one names what to change.
5. **Duplicates** (`structure.stats.duplicates`) and **leaks**: the text must be what exists when the decision is made.
   Check also that the part that decides the answer comes first in long examples (skill `typestrict-prepare-data`).
6. **The question.** It should be decided by the company's own answers (its queues, teams, products). A generic question
   (topic, sentiment) is one large models already answer well; steer it to what only this company knows.

Propose training only when this list is clean, and tell the user what you changed and why.

## 3. Start the training (costs a credit)

Tell the user the cost and their balance and **ask before spending**. Then:

```bash
typestrict run <id> --yes --watch          # --base ts-moberto-150m is the default; `typestrict encoders` lists them
```

With MCP: `quote_training`, show the quote, and `start_training` with its `confirmation` only after the user agrees.
Without credits, send the user to https://typestrict.ai/billing.

**The base model.** `ts-moberto-150m` (English and Portuguese) is the default and the right first try. For English text,
`ts-ettin-150m` is the other strong one (TS-PRGuard, which reads code changes, runs on it): when it matters, train both and
keep the version that wins. The smaller `ts-ettin-68m`, `32m` and `17m` trade accuracy for speed and memory.

**Keeping a model that does not beat Jev.** Only when the user asks for it: `--keep-if-loses` (MCP `quote_training` with
`keep_if_loses`). The model is delivered whatever the result (verdict `kept`) and the credit does not come back. Say so
before they agree.

## 4. The result

```bash
typestrict result <model>          # MCP: get_model, and get_audit for the audit link
```

`beats` means it beat Jev on the held-out examples; otherwise the credit came back (`refund`), unless the user chose to
keep it (`kept`). `retry` as next action means the training failed and the credit came back too.

**When it does not beat Jev**, read why in the comparison (`model.jev.superiority.reason`): a gain too small to be sure
of, or one answer that trails Jev. More examples of the weak answers, or another base model on the same data, are the
next tries: every version is measured on the same held-out examples, so versions compare fairly.
`typestrict encoders` lists the base models that can be picked.

## 5. Download

```bash
typestrict pull <model> --dir models/ticket-router   # the package; MCP: download_model
typestrict pull <model> --container                  # the container image, for this machine's platform
```

With the remote MCP server, `download_model` returns the package's download links: save the three files with these
names, next to each other, in the user's project.

The package (`decision_ai.json`, `tokenizer.json`, `onnx/model_8bit.onnx`) and the container image (one for
`linux/amd64`, one for `linux/arm64`; `--platform` picks the other) can be downloaded for 30 days after the training; a
refunded model has none. Then use the `typestrict-integrate` skill.

Docs: https://docs.typestrict.ai/cli (for assistants: https://docs.typestrict.ai/llms-full.txt).
