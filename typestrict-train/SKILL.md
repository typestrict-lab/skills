---
name: typestrict-train
description: Train, follow and download a typestrict decision model with the typestrict CLI or MCP server. Use when the user asks to train a model on their data with typestrict, to check how a training is going, or to get its result.
---

# Train a typestrict model

Use the typestrict MCP tools when they are available; otherwise the CLI, with `--json` when you need to read the
answer. Each step below gives both.

```bash
command -v typestrict || curl -fsSL https://raw.githubusercontent.com/typestrict-lab/cli/main/install.sh | sh
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

MCP: `create_dataset` (local file paths, `hf`, `github`, `questions`), then `wait_for_change`. Keep the dataset id.

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

**Read the proposal before spending.** An answer with few examples, a warning, or answers that overlap are the usual
reasons a model ends up refunded: fix them first (skill `typestrict-prepare-data`, "Getting the best model").

## 3. Start the training (costs a credit)

Tell the user the cost and their balance and **ask before spending**. Then:

```bash
typestrict run <id> --yes --watch          # --base ts-moberto-150m is the default; `typestrict encoders` lists them
```

With MCP: `quote_training`, show the quote, and `start_training` with its `confirmation` only after the user agrees.
Without credits, send the user to https://typestrict.ai/billing.

## 4. The result

```bash
typestrict result <model>          # MCP: get_model, and get_audit for the audit link
```

`beats` means it beat Jev on the held-out examples; otherwise the credit came back. `retry` as next action means the
training failed and the credit came back too.

**After a refund**, read why in the comparison (`model.jev.superiority.reason`): a gain too small to be sure of, or one
answer that trails Jev. More examples of the weak answers, or another base model on the same data, are the next tries:
every version is measured on the same held-out examples, so versions compare fairly, and each refunded one costs nothing.
`typestrict encoders` lists the base models that can be picked.

## 5. Download

```bash
typestrict pull <model> --dir models/ticket-router   # MCP: download_model
```

The package (`decision_ai.json`, `tokenizer.json`, `onnx/model_8bit.onnx`) can be downloaded for 30 days after the
training; a refunded model has none. Then use the `typestrict-integrate` skill.

Docs: https://docs.typestrict.ai/cli (for assistants: https://docs.typestrict.ai/llms-full.txt).
