# Sentence Intent Analysis Applied to Conversational Interfaces

*English version of the original Traditional-Chinese paper（應用於對話式介面之語句用意分析）. This is a faithful translation/adaptation of the original document for readers who don't read Chinese; the original `.docx` remains in the repository as the source of record.*

> **Note:** This document is a **research proposal** — it lays out the motivation, a literature review, the planned method, a projected schedule, and *expected* results. It is not a report of completed, measured results. The interactive `demo.html` is a **simplified illustration** of the proposed pipeline, not the full system.

---

## 1. Abstract

As mobile devices have proliferated, people are increasingly inseparable from their phones, so the interface design of human–computer interaction (HCI) systems has become very important. Some HCI systems use a minimal interface to help users get started quickly; others use intuitive icons so users can grasp the system state at a glance. Most interfaces, however, still require manual operation, which is inconvenient when the user is busy or has their hands full. If the manual steps could be removed and replaced with spoken dialogue to help users complete tasks, users would no longer need to learn a multitude of different interfaces, nor free up their hands in the middle of a busy moment — making daily life more convenient.

Today's smart voice assistants can already handle simple everyday functions (setting alarms, checking the weather, playing music, etc.), but for compositional or more complex functions they often fail, because it is hard to analyze the intent of a whole sentence. For example, *"forward the email about the undergraduate project application guidelines to Ho"* requires the system to recognize that the sentence contains **two** basic actions — search and send. Likewise *"I want to find the email Ho sent me last month"* requires the system to correctly map *"find"* to the query function and *"Ho"* and *"last month"* to the query conditions (content and received-time).

This work uses the **Gmail** application as a worked example. Building on the lab's prior research in information extraction and semantic understanding, it develops a conversational email interface that can query, send, and read email through dialogue — analyzing the user's intent from spoken dialogue and accurately providing the corresponding single or composite functions.

---

## 2. Motivation & research questions

The ubiquity of mobile devices has spawned many apps that make daily life more convenient, and these apps rely on their interface design so users can perform the services they need. This has two drawbacks. **First**, every app has its own particular interface and usage conventions, and becoming familiar with all of them is very demanding for novices or older users. **Second**, many functions are buried under some page and require several taps to reach, which is impractical in situations where the user cannot free their hands — while cooking, while driving, and so on. A spoken intelligent assistant is therefore an important research topic for the future of information technology: it lets people use services through the more natural medium of spoken dialogue, making everyday life more convenient.

Products such as Google Home and Amazon Echo already let users place calls or set alarms by voice, but these assistants still have substantial room for improvement — for instance, supporting more complex functions like sending email or checking a schedule. We therefore aim to develop a new **semantic-analysis module** to extend an assistant's capabilities.

Existing assistants typically guess the desired function by **keyword matching** and then extract the parameters for that function. An alternative is to first run a broad **named entity recognition (NER)** model to identify the types of the important pieces of information in the sentence, then use a **classifier** to determine the target function so as to fulfil the user's goal.

**Named Entity Recognition (NER)** marks words of target types within a sentence; the most common approach is sequence labeling, e.g. with a **Conditional Random Field (CRF)**. With the rise of deep learning, many papers combine recurrent or related neural architectures and achieve higher accuracy (discussed below). NER models, however, need a fair amount of labelled training data, and no suitable public dataset (for our target types, in Chinese) was readily available online. We plan to use the **Web NER Model Generation Tool** developed by the WIDM lab and prepare training data via **distant supervision (distant learning)**.

The other problem is determining the **intent / target function**. A classifier is relatively easy to train *given enough data*, and many neural-network architectures exist for classification. But the pairing of inputs to actual goals is hard to find online, so we attempt to train the classifier from limited information, using **reinforcement learning** where necessary.

The goal of this study is to let users accomplish Gmail- and Google-Calendar-related operations conversationally, simplifying the process so it can be extended later by combining other programs' APIs to further enhance the assistant.

---

## 3. Literature review

### 3.1 Classifier

Any application offers only a finite set of functions, so determining the user's intent — i.e. which function to invoke — requires a classification model. Common choices include **SVM (Support Vector Machine)** and **ANN (Artificial Neural Network)**. We tentatively choose an **MLP (Multilayer Perceptron)** because the set of functions to distinguish is very broad: an SVM might need many models (one per pair of classes, ~C(n,2)) to classify them, and the classes may not be cleanly separable as two regions in space without a correct and effective projection first. A further issue is that when a new class is added, an MLP can simply add neurons in its hidden/output layers, reset the weights, and continue training to update the model, whereas an SVM has a comparatively larger computational cost.

### 3.2 Named Entity Recognition

NER is an NLP technique whose main goal is to identify entities of specific types. An *entity* can be an abstract concept such as a time or date, or a concrete thing. For example, in *"Xiaoming is going to the park tomorrow,"* "Xiaoming" is a person's name (a **Person** entity), "tomorrow" is a **Time** entity, and "park" is a **Location**. We discuss the related algorithms **CRF, RNN, LSTM and CNN**.

**Conditional Random Field (CRF)** is an undirected probabilistic model often used for NER: each vertex represents a variable and edge values represent how closely related two variables are. Lafferty et al. (2001) published the use of CRF for NER, using this probabilistic model to determine and mark the start and end of a particular data type.

**Recurrent Neural Network (RNN)** is a network whose output can feed back as input to the same layer; its strength is capturing relationships across a longer span of text, since each character's input is related to all preceding inputs. Yao et al. (2013) used an RNN for NER and found that, with a suitable architecture, the F1 accuracy exceeded a CRF model by at least 2%. Although Vukotic et al. (2015) argued a plain RNN cannot replace CRF, Yao et al. (2014) then passed the input through an RNN and fed the RNN output into a CRF for the decision, achieving F1 about 3% higher than a plain CRF and 1% higher than a plain RNN.

**Long Short-Term Memory (LSTM)**, proposed by Hochreiter & Schmidhuber (1997), uses three sigmoid gates to decide whether to *remember*, *forget*, and *output*. Its use in language was introduced by Sundermeyer et al.; it is similar in purpose to the RNN but is an enhanced, selective-memory version. Because LSTM captures long-range dependencies, Huang et al. (2015) used LSTM+CRF and Bi-LSTM+CRF for NER with accuracy higher than other algorithms. **Bi-LSTM** (bidirectional LSTM) doubles the computation by also processing the sequence in reverse.

**Convolutional Neural Network (CNN)** repeatedly applies convolution and pooling and then feeds a classifier; it is usually applied to images. Convolution slices out parts of the input (greatly increasing data volume), after which pooling reduces the information by keeping only the important parts (e.g. max-pooling keeps the largest value in a window). Recently CNNs have been applied to NLP: Kalchbrenner et al. (2014) used a CNN for sentence/part-of-speech modelling (partially replacing parse trees), and Kim (2014) used a CNN for sentence classification. A particularly interesting application is English word-root analysis: Ma & Hovy (2016) combined this idea with Bi-LSTM and CRF, first using a CNN for a character-level word representation and then a Bi-LSTM+CRF, yielding an **LSTM-CNNs-CRF** model.

### 3.3 Pre-processing for NER — word embeddings

Before extracting the important information types from a sentence, how words are represented at the input matters. Two strongly related words should have similar representations, which motivates **word embeddings**: projecting every word into a vector space so that words of similar meaning/type lie close together; the resulting vector can then represent the word. Mikolov et al. (2013) proposed the **Continuous Bag of Words (CBOW)** and **Skip-Gram (SG)** models. CBOW uses the surrounding context words to predict the keyword (good for frequent words); Skip-Gram does the reverse, predicting the surrounding context from the keyword (accurate even for rare words, but slower). In both, the trained hidden-layer weights retain part of each word's meaning and relationships and can stand in for the original word — i.e. **word vectors (Word2Vec)**.

---

## 4. Method & steps

1. **Word-vectorization tooling.** Try a simple NER algorithm to see whether it trains successfully, then try different pre-processing to see whether it improves.
2. **The NER core.** Many NER models exist, but the harder part is generating the training data: there is no Chinese database labelled with the required types online, and different functions need different label types. The WIDM lab's **Web NER Model Generation Tool** can auto-label training data from seed entities and then train a type-specific NER model with a CRF sequence-labeling package, by finding relevant sentences from the web using known entities and auto-labelling them as supervised training data.
3. **Function determination** via a neural network. Training data here is even harder to prepare — finding similar sentences from the web under some known conditions for supervised learning. If data is insufficient, attempt to train the classifier with **reinforcement learning**.

### 4.1 Concrete steps

1. Crawl reasonably complete sentences from web forums as test data and determine which word-embedding method fits best. (Critical: if the word vectors don't represent meaning well, later steps will likely perform worse.)
2. Use the WIDM lab's **PowerPOI** tool to train CRF models for specific information types (person, time, subject, …), and try adding a Bi-LSTM architecture to compare accuracy. Accuracy is computed against the crawled data, so anomalies require adjusting the crawler.
3. Make the NER training-data search and model adjustment loop run continuously and normally; try to avoid grabbing extreme samples, whose skewed wording causes mis-classification.
4. Since several NER models must be trained, automate the previous step (remove the manual adjustment).
5. Because a sentence's key point / the user's intent is hard to analyze directly, use the NER above to extract the important information and parameters and feed them into the neural network to decide the appropriate function.
6. If data is too sparse or classification fails, use reinforcement learning with the few available samples.
7. Finally, send the important information into the relevant function to execute — mainly Gmail and Google Calendar. If the process shows no obvious error yet the key point is still not captured, consider an **attention mechanism (AM)** to pre-process the sentence.

### Projected schedule (Jul 2018 – Feb 2019)

Word-embedding selection → NER data collection & model training → NER training automation → function discrimination → integration with Google APIs.

---

## 5. Expected results

From a user's command sentence, the system should identify the important words and analyze the desired function, demonstrated on Gmail and Google Calendar. For example, given *"read the letter Xiaoming sent me yesterday,"* it should analyze the search date as "yesterday," the sender as "Xiaoming," and the function as "read." Because this technique improves on several commercial smart-home products, known-purpose sentences can be used for comparison, so in principle both intent discrimination and word-type labelling should be more accurate.

---

## 6. References

1. Lafferty J., McCallum A., Pereira F. *Conditional Random Fields: Probabilistic Models for Segmenting and Labeling Sequence Data.* Proc. 18th Int. Conf. on Machine Learning, 282–289, 2001.
2. Yao K., Zweig G., Hwang M.-Y., Shi Y., Yu D. *Recurrent Neural Networks for Language Understanding.* Interspeech, 2013.
3. Vukotic V., Raymond C., Gravier G. *Is it time to switch to word embedding and recurrent neural networks for spoken language understanding?* Interspeech, 2015.
4. Yao K., Peng B., Zweig G., Yu D., Li X., et al. *Recurrent Conditional Random Field for Language Understanding.* ICASSP, 2014.
5. Hochreiter S., Schmidhuber J. *Long Short-Term Memory.* Neural Computation, 1997.
6. Sundermeyer M., Schlüter R., Ney H. *LSTM Neural Networks for Language Modeling.* INTERSPEECH-2012, 194–197, 2012.
7. Huang Z., Xu W., Yu K. *Bidirectional LSTM-CRF Models for Sequence Tagging.* arXiv:1508.01991, 2015.
8. Kalchbrenner N., Grefenstette E., Blunsom P. *A Convolutional Neural Network for Modelling Sentences.* arXiv:1404.2188, 2014.
9. Kim Y. *Convolutional Neural Networks for Sentence Classification.* arXiv:1408.5882, 2014.
10. Ma X., Hovy E. *End-to-end Sequence Labeling via Bi-directional LSTM-CNNs-CRF.* arXiv:1603.01354, 2016.
11. Mikolov T., Sutskever I., Chen K., Corrado G. S., et al. *Distributed Representations of Words and Phrases and their Compositionality.* NIPS 2013.
