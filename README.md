# Conversational HCI — Voice-Driven Email Intent Agent

*A multilingual conversational interface for Gmail that chains speech recognition, intent classification, and named entity recognition to execute email actions from natural language commands.*

📄 **[Read the Proposal](https://github.com/woodyhoko/Conversational-human-computer-interfaced/blob/main/proposal.pdf)** | 🖼️ **[View the System Poster](https://github.com/woodyhoko/Conversational-human-computer-interfaced/blob/main/poster.pdf)** | **[▶ Pipeline Demo](demo.html)**

---

## 1. Motivation

Voice assistants (Amazon Alexa, Google Home, Apple Siri) have demonstrated the viability of natural language as a primary HCI modality for smart home control. Email — a knowledge-work task requiring compositional language understanding — presents a significantly harder challenge: the assistant must identify *what the user wants to do* (intent), *who they want to do it with* (named entities), and *what they want to say* (content extraction), then execute a multi-step action via an authenticated API.

This project builds a complete **conversational email agent** for Mandarin and English, treating the problem as a pipeline of NLP components whose composition must be robust to the ambiguities of spontaneous speech.

---

## 2. System architecture

```
User Voice Input (Mandarin / English)
        │
        ▼
[1] Speech Recognition
        │  raw transcript text
        ▼
[2] Intent Classification              → {READ, SEND, PICK, FORWARD, CONFIRM}
        │  intent label + confidence
        ▼
[3] Named Entity Recognition           → {recipient, subject, content, email_index}
        │  slot-filled semantic frame
        ▼
[4] Dialogue Manager                   → slot-filling, clarification, confirmation
        │  action specification
        ▼
[5] Gmail API Executor                 → list / read / send / forward
        │
        ▼
Text-to-Speech Response
```

---

## 3. Component design

### 3.1 Speech recognition

| Backend | Architecture | Notes |
|---|---|---|
| Google Speech API | End-to-end DNN (RNN-T) | Best accuracy; requires internet |
| Bing Speech Recognition | DNN | Microsoft Azure backend |
| Kaldi | GMM-HMM acoustic + n-gram LM | On-premise; open source |
| DeepSpeech | End-to-end CNN+RNN (CTC) | Mozilla; fully local |

For Mandarin, Google Speech and Bing outperform Kaldi substantially due to the character-level complexity of Chinese phonology (tones, syllabic structure). DeepSpeech was evaluated for privacy-preserving on-device use.

### 3.2 Intent classification

Five intent classes are defined:

| Intent | Example utterance |
|---|---|
| `READ_EMAIL` | "Read my emails from Alice" |
| `SEND_EMAIL` | "Send a message to Bob saying I'll be late" |
| `PICK_EMAIL` | "Open the third email" |
| `FORWARD_EMAIL` | "Forward this to the team" |
| `CONFIRM` | "Yes, send it" |

Four classification approaches were evaluated:

**LUIS (Microsoft):** Cloud-hosted NLU service with intent + entity extraction trained on labelled examples. Fast to deploy; requires labelled data; not fully offline-capable.

**Wit.ai (Meta):** Similar cloud NLU; open-source training interface; good for Mandarin with sufficient training data.

**Dictionary + ML:** A two-stage approach — keyword matching filters candidate intents, then a logistic regression / SVM classifier resolves ambiguities. More robust to out-of-vocabulary words than pure neural approaches on small datasets.

**State Machine:** Rule-based dialogue management with explicit transition rules. Deterministic; easily audited; struggles with novel phrasings.

The hybrid Dictionary + ML approach achieved the best trade-off between accuracy and robustness on the collected Mandarin email corpus.

### 3.3 Named entity recognition

Entity slots required: `{recipient, subject, content, email_index}`. NER is formulated as a **sequence labeling** problem: each token receives a BIO tag (`B-RECIPIENT`, `I-CONTENT`, `O`, etc.).

| Model | F1 (dev set) | Notes |
|---|---|---|
| **CRF** | 78.2 | Conditional Random Field; strong baseline for structured NER |
| **Bi-LSTM** | 81.4 | Bidirectional context; handles long-range dependencies |
| **Bi-LSTM + CRF** | **83.7** | Best: Bi-LSTM features + CRF structured output layer |
| **CNN** | 79.1 | Character-level features; robust to OOV proper nouns |

The **Bi-LSTM + CRF** architecture (Lample et al. 2016) achieves the best performance by combining bidirectional contextual encoding with CRF's ability to enforce label sequence constraints (e.g. `I-RECIPIENT` cannot follow `B-CONTENT`).

```
Input tokens → Character CNN → Word embedding → Bi-LSTM → CRF layer → BIO tags
```

### 3.4 Dialogue management

A **slot-filling finite state machine** tracks which entity slots have been populated and generates clarification requests for missing ones:

```
state: {intent, slots_filled, awaiting_confirmation}

transition rules:
  SEND_EMAIL + missing recipient  → ask "Who would you like to send to?"
  SEND_EMAIL + all slots filled   → present draft + request confirmation
  CONFIRM                         → execute Gmail API send
  FORWARD_EMAIL + missing target  → ask "Forward to whom?"
```

Confirmation before destructive actions (send, forward) is a safety requirement — the system never sends email without explicit user approval.

---

## 4. Example dialogue

```
User:   "Send an email to Alice saying I'll be late to the meeting."
System: "Sending to Alice: 'I'll be late to the meeting.' Confirm?"
User:   "Yes."
System: "Email sent."
```

```
User:   "Read my emails."
System: "You have 3 unread emails. First: from Bob, subject 'Project update'…"
User:   "Open the second one."
System: "Email from Carol: 'Can we reschedule?'…"
```

---

## 5. Files

| File | Description |
|---|---|
| `proposal.pdf` | Full research proposal — problem, related work, system design |
| `poster.pdf` | System architecture poster (conference format) |
| `應用於對話式介面之語句用意分析4.docx` | Full paper in Traditional Chinese |
| `demo.html` | Interactive pipeline visualizer |

---

## 6. References

1. G. Lample et al. "Neural Architectures for Named Entity Recognition." *NAACL 2016*.
2. J. Devlin et al. "BERT: Pre-training of Deep Bidirectional Transformers." *NAACL 2019*.
3. A. Hannun et al. "Deep Speech: Scaling up end-to-end speech recognition." *arXiv:1412.5567*, 2014.
4. A. M. Turing. "Computing Machinery and Intelligence." *Mind*, 59(236):433–460, 1950. (conceptual foundation of conversational HCI)
