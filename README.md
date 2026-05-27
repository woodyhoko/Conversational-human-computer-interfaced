# Conversational HCI — Email Intent Agent

A research project building a **conversational interface for Gmail** that understands natural-language voice commands, classifies user intent, and executes email actions — bridging speech recognition, NLU, and smart home assistant paradigms.

📄 **[Read the Proposal](https://github.com/woodyhoko/Conversational-human-computer-interfaced/blob/main/proposal.pdf)** | 🖼️ **[View the System Poster](https://github.com/woodyhoko/Conversational-human-computer-interfaced/blob/main/poster.pdf)**

---

## Overview

As Google Home, Amazon Echo, and similar voice assistants become widespread, the gap between natural language and real-world action is the key challenge. This project builds a **conversational email agent** that:

1. Accepts voice or text input (Mandarin/English)
2. Classifies intent (read email / send email / pick email / forward / confirm)
3. Extracts named entities (recipient, subject, content)
4. Executes the action via Gmail API

---

## System Architecture

```
User Voice Input
      │
      ▼
[Speech-to-Text]          Google Speech API / Kaldi / DeepSpeech
      │
      ▼
[Intent Classification]   LUIS / Wit.ai / Dictionary+ML / State Machine
      │
      ▼
[Named Entity Recognition]  CRF / Bi-LSTM / CNN
      │
      ▼
[Action Execution]         Gmail API
      │
      ├── Read email
      ├── Pick/select email
      ├── Send new email
      └── Forward email → confirm content → send
```

---

## NLP Methods Evaluated

### Intent Classification

| Method | Approach |
|---|---|
| **LUIS** (Microsoft) | Cloud NLU — intent + entity extraction |
| **Wit.ai** (Meta) | Cloud NLU with training data |
| **Dictionary + ML** | Keyword matching + classifier |
| **State Machine** | Rule-based dialogue management |

### Named Entity Recognition

The system needs to extract entities like recipient names, email subjects, and content from spoken sentences. Methods compared:

| Model | Description |
|---|---|
| **CRF** (Conditional Random Field) | Sequence labeling baseline; effective for structured NER |
| **RNN / LSTM** | Captures long-range context; outperforms CRF on F1 by 2–3% |
| **Bi-LSTM + CRF** | Best combination — bidirectional context + CRF structured output |
| **CNN** | Character-level features for OOV robustness |

### Speech Recognition Backends

| Backend | Architecture |
|---|---|
| Google Speech API | Neural network (NN) |
| Bing Speech Recognition | Neural network (NN) |
| Kaldi | GMM acoustic model |
| DeepSpeech | End-to-end NN |
| Liangstein / Wavenet | Neural TTS (synthesis for testing) |

---

## Example Dialogue Flow

```
User: "Send an email to AAA saying I'll be late."
          │
          ├── Intent: SEND_EMAIL
          ├── Entity: recipient = "AAA"
          └── Entity: content = "I'll be late"

System: "Sending to AAA: 'I'll be late.' Confirm?"
User: "Yes."
System: "Email sent."
```

---

## Research Context

Conducted as academic research in Human-Computer Interaction and NLP. The full paper (in Traditional Chinese) benchmarks the NER and intent classification models on Mandarin conversational data collected from real Gmail interaction scenarios.

---

## Files

| File | Description |
|---|---|
| `proposal.pdf` | Full research proposal — problem, related work, methods |
| `poster.pdf` | System architecture poster |
| `應用於對話式介面之語句用意分析4.docx` | Full paper (Traditional Chinese) |
