---
name: typestrict-integrate
description: Put a trained typestrict model to work in an app or service. Use when the user has a typestrict model (or its downloaded package or container image) and wants to call it from their code or run it on their servers.
---

# Integrate a typestrict model

A trained model ships in two forms, both downloadable for 30 days after the training (keep your own copy):

- **The container image**: the model behind Jev's decision API, on a CPU. The way to go for a server, in any language.
- **The package**: three files for an app that runs the model itself (Flutter, or any ONNX Runtime).

## Ask what it was trained on

`decision_ai.json` (in the package, and at `/model` in the image) lists what the model learned: a single-decision model
has a `task`, a model of several decisions a `tasks` list, each with its `question` and its `options` (or `labels`).
The model answers any question, but it is at its best on these: **ask the same question with the same options**, in the
same order. A reworded option is a question it never learned. Opening the container in a browser (`GET /`) lists them.

## On a server: the container

```bash
typestrict pull <model> --container        # this machine's platform; --platform linux/amd64 for x86 servers
docker load -i <name>-v<version>-<arch>.tar
docker run -p 8080:8080 typestrict/<name>:<version>
```

One image per platform: `linux/amd64` for x86 servers, Cloud Run, AWS Lambda and most Kubernetes clusters, `linux/arm64`
for Apple Silicon, AWS Graviton and ARM servers. The other platform runs only under emulation, slowly or not at all.

It answers `POST /v1/systemone` exactly like Jev, so code that calls Jev switches by changing the base URL:

```bash
curl http://localhost:8080/v1/systemone -H "Content-Type: application/json" -d '{
  "model": "ticket-router",
  "state": "I was charged twice. Please help.",
  "questions": {
    "team": {
      "type": "choice",
      "instructions": "Which team handles the ticket?",
      "criteria": { "billing": "Payments and invoices", "support": "Everything else" }
    }
  }
}'
```

- **Several questions in one call** when they are about the same text: the container answers them in parallel, and a
  model of several decisions was trained for exactly this.
- `GET /healthz` says `ok` once the model is loaded (use it for readiness); `GET /v1/models` names the model.
- **No network calls and no telemetry**: requests and data stay in the container. To require a key, set
  `TYPESTRICT_TOKEN` (clients send `Authorization: Bearer <token>`), or use the platform's own auth.
- **Memory**: each worker session takes about 400 MB idle and up to about 2.3 GB after reading a 4,096-token input; the
  container starts as many workers as its memory limit allows. Give it 2 GB or more.
- **Latency grows faster than the input**: a few hundred tokens answer in a fraction of a second on a CPU, a 4,096-token
  input takes seconds. Send the text the decision needs, not more.

Deploy recipes (Docker, Kubernetes, Cloud Run, Lambda): https://docs.typestrict.ai/deploy.

## In an app: the package

```text
model/
  decision_ai.json     the manifest: protocol, calibration, checksums and what it was trained on
  tokenizer.json
  onnx/model_8bit.onnx the model, about 160 MB, runs on a CPU
```

Keep the files next to each other with these exact names: `decision_ai.json` checks their SHA-256.

### Flutter (iOS, Android, web): decision_ai

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
      'billing': 'Payments and invoices', // the options exactly as in decision_ai.json
      'support': 'Everything else',
    }, instructions: 'Which team handles the ticket?'),
  },
);
answers['team']!.choice;        // the chosen option
answers['team']!.probabilities; // every option's probability
```

Follow the platform setup in the decision_ai README (iOS, Android, web).

### Other languages

The package is standard ONNX with the protocol documented in `decision_ai.json`, so any ONNX Runtime can run it. For a
server, the container already does this.

## Turn answers into actions

The pattern that works, as in [pr-guard](https://github.com/typestrict-lab/pr-guard) (a model on the CI runner that says
how many people should review a pull request):

1. **Ask every question in one call** and keep each answer's probability (`noul`, `confidence`, `probabilities`).
2. **A policy the user writes decides the action**: a threshold per question ("vulnerability above 0.7 needs a second
   reviewer"), not hard-coded in the call. Changing what is automated is editing the policy, not retraining.
3. **Deterministic checks next to the model** for what a rule decides with certainty (pr-guard checks secrets and
   destructive SQL in code). The model is for what needs judgment.
4. **Low confidence goes to a person.** The result reports accuracy over the 80% most confident answers: start by
   automating above a threshold and sending the rest to a person, then move the threshold with what you see in
   production. A threshold, not a rebalanced training, is how to trade misses for false alarms.

Docs: https://docs.typestrict.ai
