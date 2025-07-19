# Sentence Constructor Training Agent - Architecture and Design

## Functional Requirements

The Sentence Constructor is a study activity in which language learners translate sentences from English to Japanese. Our goals and requirements are:

* Direct answers are not provided; instead, learners receive feedback, guidance, and contextual hints.
* The system helps learners recognize common translation errors, such as incorrect word order, vocabulary mismatches, and grammatical issues.
* Practice is reinforced through repeated exercises, with difficulty and feedback levels tailored to the learner’s pace.

## Assumptions (Tech Uncertainty)

This system is still exploratory, and we make several key assumptions about what is technically feasible:

* Translation Competency: We assume that the AI model is capable of generating accurate Japanese translations for a wide variety of English sentences. This likely depends on access to a robust corpus of bilingual training data in both directions (EN→JP and JP→EN). Errors like hallucinated vocabulary or fabricated rules may occur, but we assume these will be rare enough not to undermine trust.
* Mistake Diagnosis: We assume the model can compare a student’s attempt to a reference translation and identify meaningful differences. Calculating distance between sentences may be straightforward, but mapping errors to specific, correctable mistakes (e.g., wrong particle use, incorrect verb tense) is less certain.
* Effective Feedback Delivery: We assume the model can generate feedback that guides learners without simply giving away the answer. Striking the right balance between helpfulness and challenge is critical for learning but difficult to tune.
* Tone and Formality Bias: We assume the AI can appropriately respond to variation in tone and formality in the source English sentence, and map it to the appropriate Japanese counterpart. Biases in training data could cause overly formal or unnatural translations.
* Progress Tracking & Difficulty Scaling: We assume it's possible to score sentence difficulty and track learner progress over time. Difficulty may involve multiple dimensions (grammar, vocabulary, syntax), requiring a way to both assess and adapt.
* Compute Feasibility & Latency: We assume that the system can run on locally provisioned infrastructure or an “AI PC” without exceeding memory or processing constraints. Inference time must be fast enough to allow for real-time interaction without frustrating delays.


## Data Stratagy

We assume the system will use an in-house, locally stored dataset of sentence pairs for evaluation and feedback. Initially, this data could be sourced from public domain texts (e.g. children's books) or licensed materials, filtered for quality and alignment. Over time, additional content could be bootstrapped to increase complexity and coverage.

Ideally, the Sentence Constructor would rely on a pre-trained, general-purpose translation model that no longer requires paired sentence data for generation. However, the paired dataset remains useful for:

* Ground-truth comparison
* Measuring student output against target translations
* Generating contextual hints

Target sentences will be stored in a local SQL database, enriched with metadata like vocabulary level, grammatical features, and overall difficulty. Student performance can also be recorded in structured tables, capturing attempts, targets, timestamps, and accuracy across multiple dimensions.

While deep learning may eventually infer student proficiency implicitly, we assume for now that tracking progress through structured, interpretable skill vectors is useful. Feedback from the system will likely be deterministic—similar errors yield similar responses—with some syntactic variation for conversational tone.

Note: Privacy and data governance considerations are out of scope for this exercise but would need to be addressed in a production environment.

## Model Strategy
We assume that a small, fine-tuned model will provide the best tradeoff between performance, size, and cost for this highly specialized task. A large general-purpose LLM would likely be unnecessary overhead unless the system expands to support additional languages or domain-specific translation (e.g., medical or scientific texts).

The model (or ensemble of models) will need to support three core capabilities:

* Translation Generation: Generate a high-quality Japanese translation of an English sentence. This is optional during normal use, but critical as a fallback when the learner is struggling or for debugging and testing.

* Attempt Analysis: Evaluate student submissions by comparing them to a reference translation or a generated gold standard, and identify common errors.

* Feedback Generation: Provide meaningful hints or scaffolding to help the learner improve their response without directly giving away the answer.

While it may be possible to split these tasks into separate specialized models (e.g., one for grading, one for feedback), we assume that a unified architecture will be simpler to deploy and maintain unless clear performance benefits are demonstrated.

For now, we assume the model can operate without access to external context or documents. However, future versions might use external translation models or corpus-based comparisons to cross-check accuracy or provide multiple example translations for ambiguous input.

Deployment is expected to be centralized: the model will reside on a local AI server and be accessed via API. This supports concurrent usage by multiple students and simplifies any future aggregation of anonymized performance data for system improvement.

## System Overview

This document outlines the conceptual design of a Sentence Constructor Training Agent, an interactive tool to help students learn to translate sentences from English to Japanese. The system prompts learners to attempt translations themselves, then provides feedback—not direct answers—to guide them toward improvement. It dynamically adjusts sentence difficulty based on student performance, aiming to keep learners in an optimal challenge zone.

We provided a conceptual diagram and architecture overview (attached), and discussed functional requirements, technical assumptions, and potential risks—particularly the difficulty of diagnosing translation errors, which may prove harder than translation itself.

We proposed a data strategy based on structured tables to store both the training corpus and student performance metrics. For training, we assumed access to previously translated texts, possibly drawn from public domain or licensed sources. Finally, we outlined a tentative model strategy centered on a stand-alone local model hosted on a central server and accessed via API calls from client applications.