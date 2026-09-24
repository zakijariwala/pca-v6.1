---
block: B10
title: AI/ML and securing AI
pillars: [security, cost, performance]
exam_guide_refs: ["1.3", "2.4", "2.5", "3.1"]
last_verified: 2026-09-24
sources:
  - https://docs.cloud.google.com/gemini-enterprise-agent-platform/vertex-ai-name-changes
  - https://docs.cloud.google.com/gemini-enterprise-agent-platform/overview
  - https://docs.cloud.google.com/vertex-ai/docs/pipelines/introduction
  - https://docs.cloud.google.com/bigquery/docs/bqml-introduction
  - https://docs.cloud.google.com/tpu/docs/intro-to-tpu
  - https://docs.cloud.google.com/ai-hypercomputer/docs/overview
  - https://docs.cloud.google.com/gemini/enterprise/docs/overview
  - https://docs.cloud.google.com/model-armor/overview
  - https://docs.cloud.google.com/sensitive-data-protection/docs/sensitive-data-protection-overview
unverified: []
---
# B10 AI/ML and securing AI

Google renamed Vertex AI to Gemini Enterprise Agent Platform after the exam guide came out. The exam guide uses "Vertex AI." Expect either name.

| Exam guide name | Current name |
|---|---|
| Vertex AI | Gemini Enterprise Agent Platform (Agent Platform) |
| Vertex AI Model Garden | Agent Platform Model Garden |
| Vertex AI Pipelines | Agent Platform Pipelines |
| Vertex AI Studio | Agent Studio |
| Vertex AI Search | Agent Search |
| Vertex AI Endpoints | Agent Platform Endpoints |
| Vertex AI Agent Engine | Agent Runtime |

## Chunk 1: The platform
Problem it solves: Training, tuning, serving, and governing models and agents in one place instead of stitching tools together.

Mental model: Agent Platform (Vertex AI) covers the lifecycle: Model Garden with over 200 foundation models (Gemini, open models, third-party models), managed training, endpoints for online inference, batch inference, a model registry, evaluation, and monitoring. Agent Studio lets you prototype prompts and agents without code; the Agent Development Kit is for code.

Exam signals: "Managed ML platform", "deploy a model behind an endpoint", "choose among foundation models."

Trap: Building a custom serving stack on GKE when a managed endpoint meets the requirement.

Pillar tie-in: Operational excellence.

Check: Where do you browse and deploy foundation models, including Gemini and open models?
<details><summary>Answer</summary>Model Garden on Agent Platform (Vertex AI Model Garden in the exam guide).</details>

## Chunk 2: Least effort first: pre-trained APIs, BigQuery ML, Gemini, custom
Problem it solves: Teams overbuild. The exam rewards the simplest option that meets the requirement.

Mental model: Climb only as far as needed.

| Step | Option | Fits |
|---|---|---|
| 1 | Pre-trained APIs (Vision, Speech, Natural Language, Translation, Video) | Standard tasks, no training data |
| 2 | Gemini or another foundation model, prompted | Generation, summarization, extraction, chat |
| 3 | BigQuery ML | SQL users, data already in BigQuery |
| 4 | Fine-tune a foundation model | Domain style or format the prompt can't reach |
| 5 | Custom training | Unique problem, own data, ML team |

BigQuery ML trains and runs models with SQL (`CREATE MODEL`) and can call Agent Platform models from SQL.

Exam signals: "No ML expertise" → pre-trained API or Gemini. "Analysts know SQL, data in BigQuery" → BigQuery ML.

Trap: Custom training to detect labels in photos, when the Vision API does it.

Pillar tie-in: Cost and operational excellence.

Check: Analysts want a churn model on data already in BigQuery, and they only know SQL. What do you recommend?
<details><summary>Answer</summary>BigQuery ML.</details>

## Chunk 3: Pipelines and MLOps
Problem it solves: Notebook experiments don't repeat. Production ML needs automated, auditable steps.

Mental model: Agent Platform Pipelines (Vertex AI Pipelines) runs ML workflows serverless, defined with Kubeflow Pipelines or TensorFlow Extended (TFX). A pipeline chains data prep, training, evaluation, and deployment, and records lineage.

Exam signals: "Automate retraining", "reproducible ML workflow", "MLOps."

Trap: Cron jobs on a VM that run training scripts.

Pillar tie-in: Operational excellence.

Check: Which two SDKs can define Agent Platform Pipelines?
<details><summary>Answer</summary>Kubeflow Pipelines and TensorFlow Extended (TFX).</details>

## Chunk 4: AI Hypercomputer, GPUs, TPUs
Problem it solves: Large training and serving jobs need accelerators and the right consumption model.

Mental model: TPUs are Google's custom ASICs for the large matrix operations in ML, with on-chip high-bandwidth memory. GPUs come through accelerator-optimized VMs. AI Hypercomputer is Google's integrated supercomputing system for AI: performance-optimized hardware, open software, and flexible consumption models. Consumption options range from on-demand to Spot to reserved capacity; Spot suits fault-tolerant training with checkpoints.

Exam signals: "Train a large model at lowest cost, can restart" → Spot accelerators with checkpointing. "TensorFlow or JAX at scale" → TPUs.

Trap: On-demand GPUs for a week-long interruptible training run.

Pillar tie-in: Cost and performance.

Check: What are TPUs?
<details><summary>Answer</summary>Google's custom ASICs built to accelerate the matrix operations in machine learning.</details>

## Chunk 5: Gemini Enterprise and Agent Search
Problem it solves: Employees need AI search and assistants over company data, without building an app.

Mental model: Gemini Enterprise is an intranet search, AI assistant, and agent platform for employees. It searches across connected company data with permission-aware results, and hosts agents; NotebookLM is part of it. Agent Search on Agent Platform (Vertex AI Search) lets developers build search into their own apps and websites.

Exam signals: "Employees search internal documents with natural language" → Gemini Enterprise. "Add semantic search to our product site" → Agent Search.

Trap: Building a custom RAG stack for internal document search that Gemini Enterprise covers.

Pillar tie-in: Operational excellence.

Check: Which offering is for employees searching company data, and which is for developers adding search to an app?
<details><summary>Answer</summary>Gemini Enterprise for employees; Agent Search on Agent Platform for developers.</details>

## Chunk 6: Securing AI: Model Armor and Sensitive Data Protection
Problem it solves: LLM apps face prompt injection, jailbreaks, harmful output, and leaks of sensitive data.

Mental model: Model Armor screens prompts before they reach the model and responses before they reach the user. It checks for malicious input such as prompt injection and jailbreaks, content safety, and sensitive data. It works with models on Google Cloud and other providers. Sensitive Data Protection (which includes Cloud DLP) discovers, classifies, and de-identifies sensitive data through masking, redaction, and tokenization, using detectors called infoTypes.

Exam signals: "Block prompt injection", "stop the chatbot leaking PII" → Model Armor. "Find PII across BigQuery before training" → Sensitive Data Protection.

Trap: Relying on the system prompt to stop prompt injection.

Pillar tie-in: Security.

Check: What does Model Armor inspect?
<details><summary>Answer</summary>LLM prompts and responses.</details>

## Chunk 7: Responsible AI and case study angles
Problem it solves: AI answers must be explainable, reviewed, and safe for the business.

Mental model: Build in human review for high-impact outputs, evaluation before release, grounding in trusted data to reduce made-up answers, and logs of prompts and decisions for audit. The case studies lean on these: content moderation and auditable AI for Altostrat, human-in-the-loop catalog review for Cymbal, and AI on regulated data for EHR and KnightMotives.

Exam signals: "Decisions must be explainable", "associates approve generated content."

Trap: Auto-publishing generated content where the case says a human reviews it.

Pillar tie-in: Security and operational excellence.

Check: A retailer generates product descriptions with AI. The case requires associates to approve them. What design element must the answer include?
<details><summary>Answer</summary>A human-in-the-loop review step before content goes live.</details>
