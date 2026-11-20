# Vietnamese Aspect-Based Sentiment Analysis with PhoBERT

![Python 3](https://img.shields.io/badge/Python-3-3776AB?style=flat-square&logo=python&logoColor=white) ![PyTorch](https://img.shields.io/badge/PyTorch-EE4C2C?style=flat-square&logo=pytorch&logoColor=white) ![PhoBERT](https://img.shields.io/badge/PhoBERT-6A5ACD?style=flat-square) ![Course · NLP](https://img.shields.io/badge/Course%20%C2%B7%20NLP-blue?style=flat-square)

## Overview

This project addresses **Aspect-Based Sentiment Analysis (ABSA)** for product reviews of beauty products on e-commerce platforms in Vietnam. Unlike conventional sentence-level sentiment analysis, it extracts specific aspects of a product (e.g., *Colour, Texture, Price, Shipping*, etc.) and classifies the user's attitude (Positive, Negative, or Neutral) toward each aspect. Solving this task supports text mining workflows by automating the collection of feedback, enabling more precise optimization of products and services. The project applies state-of-the-art large language models such as **PhoBERT**, designed for Vietnamese, and compares two machine learning architectures: **Single-Task Learning (STL)** and **Multi-Task Learning (MTL)**.

---

## Dataset

The data consists of Vietnamese reviews of cosmetic products (mainly lipstick), annotated in detail.

- **Aspects:** `stayingpower` (colour durability), `texture` (Texture), `smell` (Scent), `price` (Price), `colour` (Colour), `shipping` (Shipping), `packing` (Packaging).
  *(Note: the `others` aspect was removed during preprocessing because its label distribution is concentrated almost entirely on `neutral`, offering no distinctive value for the model.)*
- **Sentiment Labels:** Positive, Negative, Neutral.
- **Data Split:** The data is split into `Train` (augmented to 122,911 samples to improve performance), `Test` (1,623 samples), and `Validation` (1,623 samples).
- **Aspect Label Format:** The data is structured in the IOB (Inside, Outside, Beginning) format for Named Entity Recognition (NER).

---

## Workflow

### 1. Data Preprocessing

- **IOB Formatting:** Maps the words in each review to their corresponding labels to prepare the data for the Token Classification task.
- **Feature Optimization:** Removes the `others` label, which acts as noisy feature that reduces the model's focus, making the system leaner and more accurate at classifying the main aspects.
- **Data Augmentation:** Enlarges the training set (from ~12.9k to ~122.9k samples) to address label distribution imbalance and improve the model's contextual coverage.

### 2. Model Architecture

The project implements and compares two approaches based on **PhoBERT**:

#### Approach 1: Single-Task Learning (STL) - Independent Pipeline

- **Model 1 (Aspect Detection):** Formulated as a Token Classification (NER) task. PhoBERT is fine-tuned with a classification layer to predict the IOB label sequence.
- **Model 2 (Sentiment Classification):** Formulated as a Sequence Classification task. The input is created by concatenating the extracted aspect with the original sentence (e.g., `[CLS] màu sắc [SEP] Son lên màu đẹp nhưng mau trôi. [SEP]`) to provide context for sentiment prediction.

#### Approach 2: Multi-Task Learning (MTL)

- Builds a single architecture with a **Shared Encoder** (a common PhoBERT backbone) to extract semantic representations.
- Attaches two separate **Heads** to the same shared encoder:
  - *Aspect Detection Head:* A Linear layer for the Token Classification task.
  - *Sentiment Classification Head:* A Linear layer for the Sequence Classification task.
- **Optimization:** The model is trained jointly using a combined loss function (a weighted sum of the Aspect and Sentiment losses). This approach allows the model to share knowledge, generalize better, and avoid the error propagation weakness of the STL approach.

### 3. Evaluation Metrics

The system is evaluated comprehensively using the following metrics:

- **Aspect Detection:** `F1-score` (Precision, Recall) based on Exact Match, where a prediction is correct only when both the span (word range) and the type of the aspect match.
- **Sentiment Classification:** `Macro F1-score` and `Weighted F1-score`, to accurately reflect performance on imbalanced datasets.
- **End-to-End Evaluation:** The strictest and most practical metric. A prediction is considered fully correct only when the system correctly identifies **both** the aspect and the corresponding sentiment in the review.

---

> Course project — Natural Language Processing (NLP)
