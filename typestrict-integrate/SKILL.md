---
name: typestrict-integrate
description: Put a trained typestrict model to work in an app or service. Use when the user has a typestrict model (or its downloaded package) and wants to call it from their code.
---

# Integrate a typestrict model

A typestrict model is a package of three files, from `typestrict pull <model>` (or the MCP tool `download_model`):

```text
model/
  decision_ai.json     the manifest: protocol, calibration, checksums and the task it was trained on
  tokenizer.json
  onnx/model_8bit.onnx the model, about 160 MB, runs on a CPU
```

`decision_ai.json` has a `task` with the trained `question` and its `labels` and `options`. **Ask the model that same
question with those same options**: it was trained on them, and other wording measures something it never learned.

## Flutter (iOS, Android, web): decision_ai

[decision_ai](https://pub.dev/packages/decision_ai) (MIT) reads this package as it is.

```yaml
dependencies:
  decision_ai: ^1.0.0
flutter:
  assets:
    - assets/model/
    - assets/model/onnx/
```

```dart
final ai = await DecisionAI.local(); // the package in assets/model/
// or: DecisionAI.remote(ModelSource.url('https://your-cdn/model/'))
// or, once published to Hugging Face from the model page: DecisionAI.huggingFace('user/repo')

final answers = await ai.decide(
  state: ticketText,
  questions: {
    'team': const Choice({
      'billing': 'Payments and invoices', // the options exactly as in decision_ai.json's task
      'support': 'Everything else',
    }, instructions: 'Which team handles the ticket?'),
  },
);
answers['team']!.choice;        // the chosen option
answers['team']!.probabilities; // every option's probability
```

Follow the platform setup in the decision_ai README (iOS, Android, web).

## Other languages and servers

The package is standard ONNX with a documented protocol in `decision_ai.json`, so any ONNX Runtime can run it.

A ready container that answers like Jev's decision API (`POST /v1/systemone`), for Docker, Cloud Run, AWS Lambda and
Kubernetes, is coming: https://docs.typestrict.ai/deploy. When it ships, switching from Jev to the model is changing the
base URL.

## Before shipping

- Keep the package files next to each other with these exact names; `decision_ai.json` checks their SHA-256.
- Use the model's `confidence` to decide what to automate and what to send to a person.
- The files on typestrict are kept for 30 days after the training: keep your own copy.

Docs: https://docs.typestrict.ai
