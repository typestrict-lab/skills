---
name: typestrict-prepare-data
description: Prepare data to train a typestrict decision model. Use when the user wants to train a model on their files (spreadsheets, CSV, JSON, documents, logs, a zip) or a public dataset, before uploading anything to typestrict.
---

# Prepare data for typestrict

typestrict trains a small model that makes one typed decision (pick an option, yes or no, or a score) on the user's data,
and refunds the credit if it does not beat Jev, the market reference model. Good data is what makes it win.

## What the user needs to know first

- **The decision.** One question in plain words, like "Which team handles this ticket?". This becomes the `goal`.
  Several decisions on the same data are possible (up to 12): list them as `questions`.
- **The answers, if they exist.** A column with past answers (the team that handled each ticket, approved or not) is
  the best case: it becomes the labels. Without answers, the user labels a sample in the review step.

## Formats typestrict reads

- Tables: CSV, TSV, Excel (xls, xlsx, xlsm), ODS, JSON and JSON lines. One row per example.
- Documents: PDF, DOCX, HTML, plain text. One document per example.
- Logs, and zip files (opened, folders kept as context).
- Images are not used in this version.

Limits: 200 MB per file, 500 files per dataset, 10 public sources (Hugging Face datasets, GitHub repositories with a
license that allows commercial use).

## Check before uploading

1. Open the file and confirm one example per row (or per document), with the text the decision is about.
2. Find the answer column, if any, and count each answer. Warn when an answer has very few examples: the model learns
   it worse, and that answer holds back the whole model.
3. Remove columns that leak the answer from somewhere it would not exist at decision time (for example a "resolved by"
   timestamp when predicting the team).
4. Personal data does not need to be removed: typestrict masks names, emails, phone numbers, documents (CPF, CNPJ) and
   card numbers before anything else reads the files, and deletes the original. Only set `anonymize: false` when the
   user states the files hold no personal data.
5. Keep each example within the base model's context (8,192 tokens, about 30,000 characters): longer ones are refused,
   never cut.

## Getting the best model

What we measured running public datasets through typestrict:

- **Your own categories win.** The model beats Jev by far on decisions with the company's own answers (its queues,
  teams, products, intents): BANKING77 0.889 against 0.797 macro-F1, a customer-support set 0.998 against 0.771. On
  generic questions with little data (topic of a news text, type of a contract clause with a few hundred examples) Jev
  knows a lot already and the margin is thin. Frame the decision around the user's own answers.
- **Examples per answer matter more than the total.** Aim for hundreds of labeled examples per answer when the data has
  them. Our base model is pretrained on decisions, which helps most with very few examples; with many, the data decides.
- **The weakest answer sets the bar.** A model is as good as its worst answer: one answer with few examples drags the
  whole model down. An answer needs at least 5 test examples to be measured (the test examples are 20% of the
  data, or 150 the user labels when the data has no answers): merge a rare answer with its closest one, or collect
  more of it, before training.
- **Answers that do not overlap, with a description.** Each answer's description is part of what the model reads. Write
  them so a person would never hesitate between two.
- **The input the model will see in production.** Train on the text that exists at decision time, nothing that is only
  known later (see the leak check above).
- **Consistent labels.** When the user labels the test examples, the same case must get the same answer every time: the
  model is measured against those answers.

Then use the `typestrict-train` skill to send it with the typestrict CLI or MCP server and train.

Docs: https://docs.typestrict.ai (for assistants: https://docs.typestrict.ai/llms-full.txt).
