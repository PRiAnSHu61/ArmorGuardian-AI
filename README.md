# ArmorGuardian AI

**Offline, edge-native PII compliance agent** — built for the ArmorIQ Hackathon (Track 2: AI Agent for the Real World).

## The Problem

Sensitive personal data — Aadhaar numbers, medical records, bank statements, legal contracts — gets processed daily in places with zero security infrastructure: rural clinics, bank correspondents doing KYC in villages, airgapped border checkpoints, disaster-response NGOs. Every existing PII compliance tool assumes cloud connectivity, technical expertise, and a SaaS budget. None of that exists at the edge. ArmorGuardian was built to fill that gap.

## What It Does

A field officer uploads or scans a sensitive document. ArmorGuardian:

1. Reads the document via OCR
2. Automatically detects which country's law applies (India → DPDP, EU → GDPR, etc.)
3. Extracts all PII using a 3-layer detection pipeline
4. Cryptographically locks the agent's intent before touching any data
5. Redacts sensitive fields with policy enforcement
6. Generates a tamper-evident audit log mapped to specific legal articles
7. Returns the clean, compliant document — **entirely offline, in under 60 seconds**

## Architecture

A 6-node [LangGraph](https://github.com/langchain-ai/langgraph) pipeline: `router → rag → pii → armoriq → redact → trust`

- **PII extraction (3 layers):** an on-device **Gemma 2B** model (4-bit quantized via [Unsloth](https://github.com/unslothai/unsloth)) + **spaCy** NER (with a 7-layer noise filter to cut false positives) + rule-based regex as a backstop
- **RAG layer:** **ChromaDB** indexing 15 regulatory articles across GDPR, DPDP, HIPAA, PCI-DSS, UK GDPR, and APPI — jurisdiction detection and policy lookup are grounded in real legal text, not guessed
- **Security core:** the **ArmorIQ SDK** cryptographically locks the agent's intent (`capture_plan()` + `get_intent_token()` + `invoke()`) before any PII is touched — this makes prompt-injection-based data exfiltration structurally impossible, not just probabilistically unlikely
- **Tool exposure:** a custom **FastAPI + JSON-RPC 2.0** MCP server registers the agent's tools on the ArmorIQ platform, tunneled via ngrok for use from Colab
- **OCR pipeline:** **EasyOCR** + **Pytesseract**, with a PDF-to-image preprocessing step (`pdf2image`, 250 DPI) for scanned documents
- **Frontend:** a **Gradio** demo UI showing the trust report, ArmorIQ enforcement log, and a privacy Q&A panel

## Compliance Frameworks Supported

| Framework | Region |
|---|---|
| EU GDPR | Europe |
| India DPDP Act 2023 | India |
| HIPAA | USA (Healthcare) |
| PCI-DSS v4.0 | Global (Finance) |

## Tech Stack

`Python` `Gemma 2B` `Unsloth` `LangGraph` `spaCy` `ChromaDB` `SentenceTransformers` `EasyOCR` `FastAPI` `Gradio` `ArmorIQ SDK`

## The Core Idea

Traditional AI safety filters are probabilistic and can be bypassed. ArmorGuardian's ArmorIQ intent lock is cryptographic — making prompt injection structurally impossible rather than just probabilistically unlikely, while running entirely offline on hardware as modest as a Raspberry Pi.

---
