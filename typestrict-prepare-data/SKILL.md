---
name: typestrict-prepare-data
description: Prepare data to train a typestrict decision model. Use when the user wants to train a model on their files (spreadsheets, CSV, JSON, documents, logs, a zip) or a public dataset, before uploading anything to typestrict.
---

# Prepare data for typestrict

typestrict trains a small model that makes one typed decision (pick an option, yes or no, or a score) on the user's data
and measures it against Jev, the market reference model. Good data is what makes it win, and you are in the best place
to prepare it: you can open the user's files, ask them what each column means and fix what is wrong before anything is
sent. Do that work here instead of leaving it to the typestrict agent.

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

## Prepare it yourself

Build one clean table (CSV or JSON lines) and send that, not the raw export:

1. **One row per example**, with a text column holding everything the decision is made on (join subject and body, or
   the fields a person would read) and nothing else.
2. **The answer column**, if the data has past answers. Normalize it: the same answer spelled the same way (`Billing`,
   `billing ` and `Faturamento` are one answer), empty and "other" rows out unless "other" is a real answer.
3. **Count each answer** and fix the rare ones now: merge an answer with its closest one, or ask the user for more
   examples of it (see "The weakest answer sets the bar" below).
4. **Remove leaks**: columns or text that only exist after the decision (a "resolved by" field or a closing note when
   predicting the team).
5. **Remove exact duplicates**, and examples longer than 20,000 characters: longer ones are refused, never cut. A model
   of several decisions reads up to 4,096 tokens of each example (about 15,000 characters of English).
6. **Put what decides the answer first.** When the text joins several parts (a description and a diff, a subject and
   a long thread), start with the part that decides the answer. If an example is longer than what the model reads, the
   end is what it may not see, and two examples that look the same to the model with opposite answers teach it nothing.
7. **Write the decision** for `goal`: the question in plain words, and the answers with one line each on when each one
   applies. The more precise this is, the closer the agent's first proposal is to what the user wants.

Show the user the counts per answer and what you changed before sending. Personal data does not need to be removed:
typestrict masks names, emails, phone numbers, documents (CPF, CNPJ) and card numbers before anything else reads the
files, and deletes the original. Only set `anonymize: false` when the user states the files hold no personal data.

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
- **Positives and negatives that look alike.** If one answer's examples all share a style (longer, better written, from
  another source), the model learns the style and not the content. Collect the examples of every answer the same way,
  so the only difference between them is the thing being decided.
- **Do not throw data away to balance the answers.** Equalizing the counts moves where the model draws the line, not how
  well it separates the answers; the line is set later, with the confidence threshold (skill `typestrict-integrate`).
  Rare answers need more examples, not fewer of the common ones.
- **Leave to a rule what a rule decides.** A secret in the text, a forbidden word, a field that is empty: check them in
  code, next to the model, and train the model on what needs judgment.
- **Split a hard question.** A broad question ("is this change risky?") is easier to learn as a few concrete ones ("does
  it break compatibility?", "are the tests enough?"), trained together as one model of several decisions.
- **Consistent labels.** When the user labels the test examples, the same case must get the same answer every time: the
  model is measured against those answers.

Then use the `typestrict-train` skill to send it with the typestrict CLI or MCP server and train.

Docs: https://docs.typestrict.ai (for assistants: https://docs.typestrict.ai/llms-full.txt).
