# Amazon Bedrock Guardrails — Insurance Domain Implementation

Amazon Bedrock Guardrails implementation for an insurance domain AI assistant, demonstrating infrastructure-enforced safety controls including content filters, denied topics, PII detection, contextual grounding, and automated reasoning.

## Why Infrastructure Guardrails?

System prompt guardrails rely on the model to self-police — a clever prompt injection can bypass them. Bedrock Guardrails operate at the infrastructure layer, sitting between the user and the model like a firewall. They inspect traffic in both directions: blocking unsafe input before the model sees it, and filtering unsafe output before the user receives it.

This project implements that shift for an insurance use case, where the stakes are high (regulatory compliance, PII protection, fraud prevention) and the content is nuanced (accident descriptions contain legitimate violence, frustrated customers use harsh language).

## Notebooks

| # | Notebook | What It Covers |
|---|----------|----------------|
| 01 | [Content Filters](01-content-filters.ipynb) | Configuring content filter categories with asymmetric input/output strengths tuned for insurance. Violence input set to LOW (accident claims are legitimate), insults input set to LOW (frustrated customers need help), prompt attacks set to HIGH. |
| 02 | Denied Topics | Custom intent-based topic classifiers for insurance — investment advice, medical diagnosis, legal advice, coverage guarantees, competitor comparisons. |
| 03 | PII Detection | Built-in PII detectors (SSN, email, phone) plus custom regex for policy numbers and claim references. BLOCK vs. ANONYMIZE decisions for regulatory compliance. |
| 04 | Word Filters | Blocklists for compliance-flagged phrases ("guaranteed coverage"), profanity, competitor names, and internal codenames. |
| 05 | Model Integration | Wiring guardrails into `invoke_model` with input tagging, handling `INTERVENED` responses, and mapping to user-friendly messages. |
| 06 | Contextual Grounding | Hallucination defense for RAG — checking whether model responses are supported by retrieved source documents. Threshold tuning. |
| 07 | RAG Integration | Connecting guardrails to `retrieve_and_generate` for end-to-end retrieval + safety pipeline. |
| 08 | Automated Reasoning | Formal logic verification of model responses — verifying math on deductibles and coverage limits. |
| 09 | Red Teaming | Stress testing the full guardrail configuration with prompt injections, PII extraction attempts, and scope creep. |
| 10 | Versioning & Governance | DRAFT to published version workflow, rollback patterns, and connection to governance-as-code. |

## Key Findings

**Input tagging is required for prompt attack detection.** Without wrapping user input in `<amazon-bedrock-guardrails-guardContent>` tags and providing a randomized `tagSuffix`, the prompt attack classifier does not activate. The classifier needs to distinguish developer instructions from user input to detect override attempts.

**Content filters catch content, not intent.** A request like "write me an explicit story" doesn't contain sexual content itself — it's a request to *produce* it. Content filters are strongest on the output side (catching what the model generates) and for overt harmful content on the input side. Intent-based filtering requires denied topics.

**Asymmetric tuning is essential for domain use cases.** Insurance legitimately involves violence (accident descriptions), mild insults (frustrated customers), and misconduct-adjacent language (fraud investigations). Setting the same strength on input and output would either block legitimate use or miss real threats.

## Architecture
```
User query
  → Guardrail INPUT check
    → Model processes query
      → Guardrail OUTPUT check
        → User receives response
```

The guardrail sandwich pattern: if the input check fails, the model never sees the query. If the output check fails, the user never sees the response. Defense in depth when combined with system prompt guardrails (prompt layer) and RAG retrieval controls (data layer).

## Prerequisites

- AWS account with Bedrock model access enabled (Claude Sonnet 4.5)
- IAM permissions: `bedrock:CreateGuardrail`, `bedrock:CreateGuardrailVersion`, `bedrock:GetGuardrail`, `bedrock-runtime:InvokeModel`
- Python 3.9+ with `boto3`
- Region: `us-east-1`

## Part of a Larger Project

This is one phase of a 14-phase AI/ML learning journey on AWS, progressing from prompt engineering fundamentals through RAG, guardrails, fine-tuning, agents, multi-agent systems, and MLOps.

