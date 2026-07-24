# Tokens & Tokenization

Notes on how Large Language Models (LLMs) actually "read" text.

Understanding tokens and tokenization is foundational for:

* Large Language Models
* RAG pipelines
* Embeddings
* Prompt engineering
* Context window management
* API cost estimation
* Multimodal AI

---

## Table of Contents

* [What Is a Token?](#what-is-a-token)
* [What Is Tokenization?](#what-is-tokenization)
* [Tokenization Algorithms](#tokenization-algorithms)
* [Word-Level vs Character-Level vs Subword Tokenization](#word-level-vs-character-level-vs-subword-tokenization)
* [Why Tokens Matter](#why-tokens-matter)
* [Context Window Limits](#1-context-window-limits)
* [Cost](#2-cost)
* [Latency](#3-latency)
* [Language Matters](#4-language-matters)
* [Try It Yourself](#try-it-yourself)
* [Practical Takeaways](#practical-takeaways-for-building-with-llms)
* [Special Tokens](#special-tokens)
* [Vocabulary Size](#vocabulary-size)
* [How Tokenizers Are Trained](#how-tokenizers-are-trained)
* [Byte-Level Fallback](#byte-level-fallback)
* [Tokenization Quirks and Model Behavior](#tokenization-quirks-that-affect-model-behavior)
* [Multimodal Tokenization](#multimodal-tokenization)
* [Complete Tokenization Pipeline](#complete-tokenization-pipeline)
* [Further Exploration](#further-exploration)
* [Key Takeaways](#key-takeaways)

---

# What Is a Token?

A **token** is the basic unit of text that an LLM reads and generates.

A token is not necessarily:

* A complete word
* A single character

It is usually somewhere in between.

Examples of what can be a single token:

| Token Type              | Example                    |
| ----------------------- | -------------------------- |
| Common word             | `cat`                      |
| Word fragment           | `token` + `ization`        |
| Character               | Rare symbols or characters |
| Punctuation             | `,` `.` `!`                |
| Word with leading space | `" the"`                   |

For example:

```text
tokenization
```

May be split as:

```text
["token", "ization"]
```

A common word may remain as a single token:

```text
cat
```

```text
["cat"]
```

A word with a leading space may be treated differently:

```text
"the"
" the"
```

These can potentially be different tokens depending on the tokenizer.

---

## Rule of Thumb

For English text:

```text
1 token ≈ 4 characters
1 token ≈ 0.75 words
100 tokens ≈ 75 words
1,000 tokens ≈ 750 words
```

However, these are only rough estimates.

Token counts depend on:

* The language
* The tokenizer
* The vocabulary
* The specific text

---

# What Is Tokenization?

**Tokenization** is the process of converting raw text into tokens and then into numeric IDs that the model can process.

Example:

```text
"I love Kerala"
```

```text
        ↓
    Tokenize
        ↓
["I", " love", " Kerala"]
        ↓
  Convert to IDs
        ↓
[40, 3021, 43978]
        ↓
   Feed into Model
```

The model does not directly process:

```text
"I love Kerala"
```

Instead, it processes numerical token IDs.

Each token ID is mapped to a vector called an **embedding**.

Conceptually:

```text
Text
  ↓
Tokens
  ↓
Token IDs
  ↓
Embeddings
  ↓
Transformer Model
```

The model processes these numerical representations mathematically.

---

# Tokenization Algorithms

Several tokenization algorithms are commonly used.

| Method            | How It Works                                      | Commonly Used By               |
| ----------------- | ------------------------------------------------- | ------------------------------ |
| **BPE**           | Repeatedly merges frequent adjacent pairs         | GPT-family models              |
| **WordPiece**     | Uses likelihood/probability-based subword merging | BERT                           |
| **SentencePiece** | Treats raw text as a stream of symbols            | T5, LLaMA, multilingual models |

---

## BPE — Byte Pair Encoding

BPE begins with smaller units and repeatedly merges frequently occurring combinations.

For example:

```text
p l a y i n g
```

The tokenizer may learn:

```text
p + l → pl
pl + a → pla
pla + y → play
i + n + g → ing
```

Eventually:

```text
playing
```

May become:

```text
["play", "ing"]
```

Instead of:

```text
["p", "l", "a", "y", "i", "n", "g"]
```

---

## WordPiece

WordPiece is similar to BPE but uses probability and likelihood-based criteria when deciding which pieces to merge.

It is commonly associated with BERT-style models.

---

## SentencePiece

SentencePiece treats text as a raw stream of symbols.

It can process text without relying on spaces as word boundaries, making it useful for:

* Multilingual models
* Languages without spaces between words
* General-purpose tokenization

Examples include models such as T5 and many LLaMA-based models.

---

# Word-Level vs Character-Level vs Subword Tokenization

There are three broad approaches to tokenization.

## Word-Level Tokenization

Example:

```text
"I love Python"
```

```text
["I", "love", "Python"]
```

### Advantages

* Easy to understand
* Short sequences

### Disadvantages

* Requires a huge vocabulary
* Struggles with rare words
* Cannot easily handle unseen words
* Misspellings can cause problems

---

## Character-Level Tokenization

Example:

```text
"cat"
```

```text
["c", "a", "t"]
```

### Advantages

* Very small vocabulary
* Can represent almost any text

### Disadvantages

* Very long sequences
* Less efficient
* More difficult to capture semantic meaning

---

## Subword Tokenization

Subword tokenization is the practical middle ground.

Common words can remain whole:

```text
"computer"
```

Rare words can be split:

```text
"unbelievable"
```

```text
["un", "believ", "able"]
```

This gives models:

* A manageable vocabulary
* Reasonable sequence lengths
* The ability to handle rare and unseen words

For example, a place name such as:

```text
Thiruvananthapuram
```

might be split into multiple pieces:

```text
["Thi", "ruvan", "anthapuram"]
```

The exact result depends on the tokenizer.

---

# Why Tokens Matter

Tokens affect several important aspects of working with LLMs.

---

## 1. Context Window Limits

Every model has a maximum context window.

The context window includes:

```text
Input Tokens + Conversation History + Output Tokens
```

For example:

```text
Context Window = 200,000 tokens
```

If the total input and output exceed this limit:

* Older content may be removed
* The request may fail
* The model may not be able to process the entire input

Conceptually:

```text
System Prompt
      +
User Message
      +
Conversation History
      +
Retrieved RAG Documents
      +
Model Response
      =
Context Window
```

This is especially important when designing:

* RAG systems
* Long system prompts
* Chat applications
* Document analysis systems

---

## 2. Cost

Many LLM APIs charge based on token usage.

Pricing is usually separated into:

```text
Input Tokens
+
Output Tokens
```

A rough cost formula is:

```text
Total Cost =
(Input Tokens × Input Rate)
+
(Output Tokens × Output Rate)
```

Output tokens are often more expensive than input tokens.

---

## 3. Latency

LLMs typically generate output autoregressively.

This means the model generates something like:

```text
Token 1
  ↓
Token 2
  ↓
Token 3
  ↓
Token 4
```

More output tokens generally means:

* More computation
* Longer generation time
* Higher latency

Therefore, reducing unnecessary output can improve response speed.

---

## 4. Language Matters

Different languages can require different numbers of tokens to represent the same amount of information.

Tokenizers are often trained on large amounts of English text.

As a result, languages such as:

* Malayalam
* Hindi
* Tamil
* Telugu
* Other Indian languages

may sometimes require more tokens for the same semantic content compared to English.

This can affect:

* API costs
* Context window usage
* RAG performance
* Translation workflows

> Token efficiency depends heavily on the language and tokenizer.

---

# Try It Yourself

Consider the sentence:

```text
"Claude's tokenizer isn't perfect."
```

A tokenizer might produce something similar to:

```text
["Claude", "'s", " token", "izer", " isn", "'t", " perfect", "."]
```

This is:

```text
8 tokens
```

for a sentence containing approximately:

```text
5 words
```

Contractions and less-common words can be split into multiple pieces.

The exact result varies depending on the tokenizer.

---

# Practical Takeaways for Building with LLMs

## Prompt Length Budget

Understand your model's context window before designing:

* Long system prompts
* Complex agent instructions
* RAG pipelines
* Large conversation histories

---

## Chunking for RAG

When splitting documents for retrieval, token-based chunking is often more predictable than character-based chunking.

For example:

```text
Document
   ↓
Split into chunks
   ↓
500 tokens per chunk
   ↓
Generate embeddings
   ↓
Store in Vector Database
```

This helps ensure retrieved chunks fit within the model's context window.

---

## Cost Estimation

A rough estimation formula:

```text
Cost =
(Input Tokens × Input Price)
+
(Output Tokens × Output Price)
```

Token counting is important for production applications.

---

## Non-English Applications

If building applications for languages such as Malayalam or Hindi:

```text
Same Meaning
      ↓
Different Language
      ↓
Potentially Different Token Count
```

Therefore, applications should account for potentially higher token usage.

---

# Advanced Topics

## Special Tokens

Tokenizers reserve special tokens for purposes beyond regular text.

### `<BOS>` — Beginning of Sequence

Marks the beginning of a sequence.

```text
<BOS> I love Python
```

---

### `<EOS>` — End of Sequence

Marks the end of a sequence.

```text
I love Python <EOS>
```

---

### `<PAD>` — Padding

Padding allows sequences of different lengths to be processed together.

```text
[I, love, Python, <PAD>, <PAD>]

[I, love, Python, very, much]
```

The model is instructed to ignore the padding tokens.

---

### `<UNK>` — Unknown Token

Represents an unknown or out-of-vocabulary token.

Modern subword and byte-level tokenizers rarely need this, but the concept still exists.

---

### System and Role Tokens

Chat models may use special tokens to represent conversation structure:

```text
<|system|>
You are a helpful assistant.

<|user|>
What is Python?

<|assistant|>
Python is a programming language.
```

These tokens help the model understand:

* Who is speaking
* Which content is an instruction
* Where a message starts and ends
* Where the assistant should respond

---

# Vocabulary Size

A tokenizer has a fixed vocabulary.

Typical vocabulary sizes may range from:

```text
32K → 200K+ tokens
```

The vocabulary is selected during tokenizer training.

Changing the vocabulary usually requires retraining or rebuilding the tokenizer and may also require changes to the model.

---

## Larger Vocabulary

A larger vocabulary can produce shorter sequences.

For example:

```text
"internationalization"
```

With a smaller vocabulary:

```text
["inter", "national", "ization"]
```

With a larger vocabulary:

```text
["internationalization"]
```

---

## Trade-Off

Every token requires an embedding vector.

For example:

```text
Vocabulary Size = 50,000
Embedding Dimension = 4,096
```

The embedding table contains:

```text
50,000 × 4,096
```

parameters.

### Larger Vocabulary

**Advantages:**

* Shorter sequences
* More efficient representation of common words

**Disadvantages:**

* Larger embedding table
* More parameters
* Higher memory requirements

Therefore, vocabulary size is an important model design decision.

---

# How Tokenizers Are Trained

Tokenizers are trained before the LLM itself.

The process looks like:

```text
Large Text Corpus
        ↓
Train Tokenizer
        ↓
Create Vocabulary
        ↓
Tokenize Training Data
        ↓
Train LLM
```

The tokenizer scans a large corpus and learns common patterns.

For example:

```text
play
playing
played
player
```

May lead to common tokens such as:

```text
play
ing
ed
er
```

The important point is:

> Tokenizer training is a separate process from LLM training.

The tokenizer is not continuously learning new tokens during inference.

---

# Byte-Level Fallback

Modern byte-level tokenizers can represent almost any input.

For example:

```text
xqz987🚀𓂀
```

Even if this exact sequence was never seen during training, it can be broken down into byte-level representations.

Conceptually:

```text
Unknown Input
      ↓
Raw Bytes
      ↓
Byte-Level Tokens
      ↓
Token IDs
```

This allows tokenizers to process:

* Rare words
* Misspellings
* Emojis
* Unseen scripts
* Unusual symbols
* Random strings

without necessarily using `<UNK>`.

---

# Tokenization Quirks That Affect Model Behavior

Tokenization can directly influence some common LLM weaknesses.

---

## Letter-Counting Failures

Consider:

> How many `r`s are in `strawberry`?

The model might receive something like:

```text
["straw", "berry"]
```

rather than:

```text
s t r a w b e r r y
```

The model must reconstruct the individual letters from larger token chunks.

This can make exact character counting difficult.

---

## Inconsistent Arithmetic on Large Numbers

Numbers may be split inconsistently.

For example:

```text
1234
```

might become:

```text
["12", "34"]
```

while another context may produce:

```text
["1", "234"]
```

The tokenizer does not necessarily understand mathematical digit boundaries.

Therefore, LLMs may struggle with:

* Large-number arithmetic
* Digit-by-digit operations
* Exact counting

For precise calculations, external tools such as calculators or code are often more reliable.

---

## Token Counts Are Not Comparable Across Models

Different models use different tokenizers.

Therefore:

```text
Same Sentence
      ↓
Tokenizer A → 10 tokens
Tokenizer B → 13 tokens
Tokenizer C → 8 tokens
```

Token counts should always be interpreted relative to a specific tokenizer.

---

# Multimodal Tokenization

Tokenization is not limited to text.

Modern multimodal models can process:

* Text
* Images
* Audio
* Video

---

## Image Tokenization

Images can be divided into patches:

```text
+--------+--------+--------+
| Patch  | Patch  | Patch  |
+--------+--------+--------+
| Patch  | Patch  | Patch  |
+--------+--------+--------+
```

Conceptually:

```text
Image
  ↓
Image Patches
  ↓
Visual Representations
  ↓
Visual Tokens
```

---

## Audio Tokenization

Audio can be converted into features or discrete codes:

```text
Audio Waveform
      ↓
Spectrogram / Features
      ↓
Audio Tokens
```

---

## Video Tokenization

Video can be processed as a sequence of frames:

```text
Video
  ↓
Frames
  ↓
Image Patches
  ↓
Visual Tokens
```

A multimodal model may process something conceptually similar to:

```text
[Text] [Text] [Image] [Image] [Audio] [Text]
```

This allows the model to reason across multiple modalities.

---

# Complete Tokenization Pipeline

A simplified LLM pipeline looks like this:

```text
Human Input
     ↓
Tokenizer
     ↓
Tokens
     ↓
Token IDs
     ↓
Embeddings
     ↓
Transformer
     ↓
Next Token Prediction
     ↓
Predicted Token
     ↓
Repeat
     ↓
Token Decoder
     ↓
Human-Readable Output
```

Example:

```text
"Hello, how are you?"
```

May become:

```text
["Hello", ",", " how", " are", " you", "?"]
```

Then:

```text
[15496, 11, 703, 527, 345, 30]
```

These IDs are converted into embeddings and processed by the Transformer.

The model then predicts the next token.

This process continues until:

* An end-of-sequence token is generated
* A maximum output limit is reached
* Another stopping condition occurs

---

# Tokens in RAG and Embeddings

Tokenization is especially important in RAG systems.

A typical RAG pipeline looks like:

```text
Documents
    ↓
Chunking
    ↓
Token Count
    ↓
Embeddings
    ↓
Vector Database
    ↓
Semantic Search
    ↓
Retrieved Chunks
    ↓
LLM Context Window
    ↓
Generated Answer
```

Poor token management can lead to:

* Context overflow
* Higher API costs
* Too many irrelevant chunks
* Reduced answer quality

A practical RAG system should consider:

```text
Chunk Size
+ Chunk Overlap
+ Embedding Model Limits
+ LLM Context Window
```

---

# Further Exploration

Try tokenizers yourself to understand how text is split.

Useful tools include:

* [OpenAI `tiktoken`](https://github.com/openai/tiktoken)
* Anthropic tokenizer tools
* Tokenizer tools provided by Hugging Face models

You can compare how the same sentence is tokenized differently by:

* BPE
* WordPiece
* SentencePiece
* Byte-level tokenizers

---

# Key Takeaways

* A token is the basic unit processed by an LLM.
* Tokens can be words, subwords, characters, punctuation, or bytes.
* Tokenization converts text into token IDs.
* Token IDs are converted into embeddings before entering the Transformer.
* BPE, WordPiece, and SentencePiece are common tokenization approaches.
* Subword tokenization provides a practical balance between word-level and character-level tokenization.
* Token count affects context limits, cost, latency, and RAG design.
* Different languages can require different numbers of tokens.
* Special tokens provide structure for sequences and conversations.
* Vocabulary size affects sequence length and model size.
* Tokenizers are trained before the LLM and do not normally learn new tokens during inference.
* Byte-level tokenization allows models to handle rare and unseen inputs.
* Tokenization can influence model weaknesses in letter counting and arithmetic.
* Token counts cannot be directly compared between models without considering their tokenizers.
* Images, audio, and video can also be converted into token-like representations.
* Understanding tokenization is essential for building efficient LLM, RAG, and AI applications.

---

## The Big Idea

> **An LLM does not directly process words, sentences, images, or numbers in their original form. These inputs are converted into tokens or token-like representations, transformed into numerical vectors, processed mathematically by the model, and finally converted back into human-readable output.**

The simplified mental model is:

```text
Human Language
      ↓
Tokenization
      ↓
Token IDs
      ↓
Embeddings
      ↓
Transformer
      ↓
Predicted Tokens
      ↓
Decoded Text
```

Tokenization is therefore not just a preprocessing step.

It directly influences:

* Model efficiency
* Context window usage
* API cost
* RAG chunking
* Embedding workflows
* Multilingual applications
* Model behavior and limitations

Understanding tokens is one of the first steps toward understanding how Large Language Models actually work.
