---
name: typestrict-train
description: Train, follow and download a typestrict decision model with the typestrict CLI or MCP server. Use when the user asks to train a model on their data with typestrict, to check how a training is going, or to get its result.
---

# Train a typestrict model

Use the typestrict MCP tools when they are available (`list_trainings`, `create_dataset`, `get_training`,
`wait_for_change`, `structure_dataset`, `quote_training`, `start_training`...). Otherwise use the CLI, with `--json`
when you need to read the answer:

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

`--watch` follows it until it needs the user and prints each step. Keep the dataset id it prints.

## 2. Follow next_action

`typestrict status <id> --json` returns `step` and `next_action`. Act on it:

| `next_action` | Do |
| --- | --- |
| `wait` | `typestrict status <id> --watch` (or the MCP tool `wait_for_change`). |
| `structure` | `typestrict structure <id>`, or `typestrict suggest <id>` to see decisions first. |
| `pick_decisions` | Show the suggestions (`typestrict review <id>`), then `typestrict structure <id> --question "..."`. |
| `label_examples` | Ask the user to run `typestrict label <id>` (interactive), or with MCP ask each example (`next_example_to_label`, `label_example`). |
| `start_training` | Go to step 3. |
| `start_over` | The files could not be read: show the error and send fixed files. |
| `expired` | The data expired (90 days without use): send it again. |

Before training, show the user the agent's proposal (`typestrict review <id>`): the question, the answers and how many
examples each has, and the warnings. To change it: `typestrict structure <id> --feedback "..."`.

## 3. Start the training (costs a credit)

Tell the user the cost and their balance and **ask before spending**. Then:

```bash
typestrict run <id> --yes --watch          # --base ts-moberto-150m is the default; `typestrict encoders` lists them
```

With MCP: `quote_training`, show the quote, and `start_training` with its `confirmation` only after the user agrees.
Without credits, send the user to https://typestrict.ai/billing.

## 4. The result

```bash
typestrict result <model>
```

`beats` means it beat Jev on the held-out examples; otherwise the credit came back. `retry` as next action means the
training failed and the credit came back too.

## 5. Download

```bash
typestrict pull <model> --dir models/ticket-router
```

The package (`decision_ai.json`, `tokenizer.json`, `onnx/model_8bit.onnx`) can be downloaded for 30 days after the
training; a refunded model has none. Then use the `typestrict-integrate` skill.

Docs: https://docs.typestrict.ai/cli (for assistants: https://docs.typestrict.ai/llms-full.txt).
