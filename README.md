# ERC-8004 Agent Benchmarking & Reputation Platform

**Decentralized Benchmarking and Reputation Ranking System for AI Agents based on the ERC-8004 Standard**
Bachelor Thesis — Hanoi University of Science and Technology (SOICT), Global ICT Program
Author: [Hoàng Khải Mạnh](https://github.com/StrongDZ)

This repo is the project overview / index. The actual code lives in the three repos linked below.

## Overview

Autonomous agents on Ethereum-compatible chains increasingly discover one another and exchange tasks and payments, yet a client has no neutral way to decide which agent to trust. [ERC-8004](https://eips.ethereum.org/EIPS/eip-8004) standardizes on-chain registries for agent identity and reputation but deliberately leaves the scoring of feedback off-chain — so existing block explorers and directories only expose raw, semantically ambiguous feedback, with no basis for a fair comparison across agents.

This project designs, implements, and evaluates a decentralized reputation-ranking platform for ERC-8004 agents, scoped to the Identity and Reputation registry paths. It interprets public, on-chain reputation signals into comparable trust rankings, benchmarking agents by their accumulated reputation evidence rather than by direct evaluation of task-execution capability.

An event-driven backend continuously crawls five EVM chains, decodes Identity and Reputation registry events, resolves off-chain metadata, and classifies each piece of feedback into one of several semantic categories using a rule-based cascade with a language-model fallback. A reputation engine combines low-latency incremental updates with periodic authoritative replays: it weights each piece of feedback by the quality of its supporting evidence rather than an unreliable price signal, and bounds an agent's reputation with a Bayesian confidence factor so that scarce or unsubstantiated feedback is discounted. Per-agent reputation, a five-component composite score, and a WalletTrust credibility score for feedback-submitting wallets drive an interactive real-time dashboard with leaderboards, agent profiles, and a live event feed.

**Scale:** 1.25M+ on-chain events processed · 243,000+ agents indexed · 321,000+ feedback records classified · 5 EVM chains.

## Architecture

```
EVM chains (×5) → indexer → Redpanda (raw event log) ─┬→ event-decoder → RabbitMQ → downstream workers
                                                        │                              (uri-bootstrap, wallet-enrich,
                                                        │                               trustrank, …)
                                                        └→ bronze-archiver → MinIO/S3, partitioned Parquet (raw archive)
                                                                                        ↘
                                                                     API (Go) subscribes & WS fan-out
                                                                                        ↘
                                                        Next.js dashboard — leaderboard, agent profiles, live feed

Feedback → rule cascade (at ingest) ──────────────────────────────────┐
        └→ unresolved → feedback-others → classifier cascade           │
                         (per-tag SVM / embedding-cosine / kNN / LLM,   │
                          via erc-8004-ai-service)                     ↓
                                                    feedback-grader (low-latency incremental update)
                                                                        ↓ reconciled each cycle by
                                                    score-refresh (periodic authoritative replay)
                                                                        ↓
                        5-component composite score — Reputation · Adoption · Services · Publisher · Compliance
                        (Publisher weighted by WalletTrust — a reviewer-credibility multiplier, 0.40–1.0)
```

## Repositories

| Repo | Role | Stack |
|---|---|---|
| [erc-8004-benchmarking-be](https://github.com/StrongDZ/erc-8004-benchmarking-be) | EVM event indexing & decoding, reputation/TrustRank engine, REST + WebSocket API | Go, MongoDB, Redis, RabbitMQ, Redpanda, MinIO |
| [erc-8004-benchmarking-fe](https://github.com/StrongDZ/erc-8004-benchmarking-fe) | Realtime leaderboard, agent profiles, live event feed | Next.js 14, TypeScript |
| [erc-8004-ai-service](https://github.com/StrongDZ/erc-8004-ai-service) | LLM feedback-classification fallback + classifier research notebooks | Python, FastAPI, Ollama |

The full written thesis, defense deck, and supporting research artifacts are maintained separately and aren't public yet.

## Research contributions

- Extensive offline classifier research (35+ benchmark/pipeline scripts) spanning rule cascades, per-tag SVM, embedding/FAISS retrieval, LLM comparisons, and ModernBERT fine-tuning with a data-efficiency study — quantifying the accuracy/latency/cost trade-offs behind the production design.
- A parameter sensitivity study identifying which scoring constants most influence the resulting rankings.
- Validation Registry integration is scoped as future work.

## Author

**Hoàng Khải Mạnh** — Global ICT Program, Hanoi University of Science and Technology (HUST)
[GitHub](https://github.com/StrongDZ) · [hoanmanh04@gmail.com](mailto:hoanmanh04@gmail.com)
