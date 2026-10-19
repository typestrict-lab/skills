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
   it worse, and the refund calculation checks the weakest answer too.
3. Remove columns that leak the answer from somewhere it would not exist at decision time (for example a "resolved by"
   timestamp when predicting the team).
4. Personal data does not need to be removed: typestrict masks names, emails, phone numbers, documents (CPF, CNPJ) and
   card numbers before anything else reads the files, and deletes the original. Only set `anonymize: false` when the
   user states the files hold no personal data.
5. Keep each example within the base model's context (8,192 tokens, about 30,000 characters): longer ones are refused,
   never cut.

Then use the `typestrict-train` skill to send it with the typestrict CLI or MCP server and train.

Docs: https://docs.typestrict.ai (for assistants: https://docs.typestrict.ai/llms-full.txt).
