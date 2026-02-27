# Expert Advisor Domains — Reference Framework

> **This file is loaded on-demand only.** It is referenced from the `agent-team-builder` skill during the Expert Agent Build Protocol (Extension B). Do not load it at skill startup. Load it when the user confirms they want an expert advisor built.
>
> **How to use this file:** Each domain section below is a structural framework and vocabulary foundation. It is NOT a complete knowledge base. The agent building the expert must layer 3–5 passes of deep web research on top of this framework to produce a practitioner-grade knowledge base file. This file ensures the research is structured correctly and that important concepts aren't missed. The output knowledge base should be 300–500+ lines depending on depth requested.

---

## Domain 1: Finance / FinTech

### What the Expert Needs to Reason About

A Finance Expert Advisor is NOT a general economics assistant. It reasons about **specific quantitative decisions**: position sizing, risk exposure, signal interpretation, regulatory compliance triggers, and execution quality. If the project involves money moving — or decisions that affect how money moves — this expert is relevant.

### Core Vocabulary (must be in the knowledge base)

**Market Structure**
- Order book mechanics: bid/ask spread, market depth, Level 1 vs Level 2 data, time-and-sales (tape)
- Order types: market, limit, stop, stop-limit, trailing stop, MOO/MOC (market-on-open/close), VWAP orders, TWAP orders, iceberg orders
- Execution venues: lit exchanges (NYSE, NASDAQ), dark pools, ATS (Alternative Trading Systems), internalizers
- Market microstructure: price impact, market impact cost, slippage, adverse selection, informed vs uninformed flow

**Options and Derivatives**
- Greeks: Delta (directional exposure), Gamma (rate of delta change — critical for near-expiry), Theta (time decay — negative for long positions), Vega (volatility sensitivity), Rho (interest rate sensitivity)
- Options flow analysis: unusual options activity (UOA), gamma exposure (GEX), delta-adjusted open interest (OI), put/call ratios by strike vs overall, sweep orders (aggressive, cross-exchange), block trades
- Implied volatility (IV) vs historical/realized volatility (HV/RV): IV rank (IVR), IV percentile (IVP), volatility surface, term structure (contango vs backwardation in VIX term structure)
- Key expiration mechanics: pin risk, max pain, gamma squeeze, options expiration (monthly OPEX, weekly, 0DTE)

**Risk Management**
- Value at Risk (VaR): parametric, historical simulation, Monte Carlo — each has failure modes (all fail in fat tails)
- Expected Shortfall (ES) / Conditional VaR (CVaR): measures expected loss beyond VaR threshold; superior to VaR for tail risk
- Position sizing: Kelly Criterion (full Kelly over-bets — use half-Kelly or fractional Kelly), fixed fractional, fixed ratio
- Drawdown metrics: maximum drawdown (MDD), drawdown duration, Calmar ratio (return/MDD), Ulcer Index
- Correlation and concentration risk: sector concentration, beta-adjusted exposure, correlation breakdown in crises
- Factor exposure: Fama-French factors (market beta, size/SMB, value/HML), momentum (MOM), quality, low-volatility

**Quantitative Metrics**
- Return metrics: CAGR (compound annual growth rate), absolute return, risk-adjusted return
- Risk-adjusted: Sharpe ratio (return/volatility — assumes normal distribution, misleading for skewed strategies), Sortino ratio (return/downside deviation — better for asymmetric payoffs), Omega ratio
- Alpha vs beta: alpha is risk-adjusted excess return vs benchmark; beta is market sensitivity
- Information ratio (IR): active return / tracking error — measures consistency of alpha generation

**Regulatory and Compliance**
- Pattern Day Trader (PDT) rule: 4+ day trades in 5 days in a margin account under $25k → restricted
- Wash sale rule: selling a security at a loss and buying substantially identical within 30 days — loss disallowed
- Short sale regulations: locate requirements, Reg SHO, threshold securities, naked short selling prohibition
- Reporting: Form 8949, Schedule D, PFIC rules for foreign funds, constructive sales
- SEC/FINRA: Reg NMS (best execution), Reg FD (fair disclosure), FINRA Rule 4210 (margin)
- MiFID II (EU): best execution, transaction reporting, unbundling research from execution costs

### Decision Frameworks for the Knowledge Base

**Signal Quality Assessment:**
If `options_volume > 2x_average_OI AND trade_is_sweep AND IV_expanding`: high conviction signal.
If `put/call_skew > 2 standard_deviations AND GEX_negative`: downside hedging or directional bearish bet.
Distinguish: dealer hedging (creates feedback loops) vs speculative directional bets.

**Risk Sizing Rule:**
Never size a position where a 2-standard-deviation adverse move creates a loss > 1–2% of total portfolio. Adjust for illiquidity (widen the expected move for thinly traded securities).

**When NOT to Trust Standard Metrics:**
- Sharpe ratio is meaningless for options strategies (non-normal distribution)
- VaR fails during correlation breakdowns (2008, March 2020 — everything correlates to 1)
- Backtested results with < 30 trades are statistically unreliable regardless of Sharpe ratio

---

## Domain 2: Cybersecurity

### What the Expert Needs to Reason About

A Cybersecurity Expert Advisor reasons about **threats, controls, vulnerabilities, and risk tradeoffs** — not just "is this secure?" but "what is the attack surface, what is the likelihood of exploitation, what is the impact, and what is the cost-effective control?" It must understand the attacker's perspective as well as the defender's.

### Core Vocabulary (must be in the knowledge base)

**Threat Modeling**
- STRIDE: Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege — structured threat enumeration
- PASTA (Process for Attack Simulation and Threat Analysis): attacker-centric, business-risk-focused
- Attack trees: hierarchical decomposition of how an attacker achieves a goal
- Threat actors: nation-state (APT), organized crime, hacktivists, insiders, opportunistic — each has different capabilities and motivations
- MITRE ATT&CK framework: structured taxonomy of adversary tactics (Initial Access, Execution, Persistence, Privilege Escalation, Defense Evasion, Credential Access, Discovery, Lateral Movement, Collection, Exfiltration, Command and Control, Impact)

**OWASP Top 10 (Web Application)**
1. Broken Access Control — most common; IDOR, privilege escalation, missing function-level access control
2. Cryptographic Failures — weak ciphers (DES, RC4, MD5 for passwords), no encryption in transit, improper key management
3. Injection — SQL, LDAP, OS command, XPath; parameterized queries / prepared statements prevent SQL injection
4. Insecure Design — missing threat model, no defense-in-depth, no rate limiting by design
5. Security Misconfiguration — default creds, unnecessary features enabled, verbose error messages in prod
6. Vulnerable and Outdated Components — unpatched libraries, dependencies with known CVEs
7. Identification and Authentication Failures — weak passwords, no MFA, broken session management
8. Software and Data Integrity Failures — CI/CD pipeline attacks, unsigned updates, insecure deserialization
9. Security Logging and Monitoring Failures — no logging, logs not monitored, no alerting on anomalies
10. SSRF (Server-Side Request Forgery) — attacker causes server to make requests to internal/external resources

**Authentication and Authorization**
- Authentication: who you are. Authorization: what you can do. Confusion between the two is a common source of vulnerabilities.
- OAuth 2.0 flows: Authorization Code (with PKCE for public clients — required), Implicit (deprecated), Client Credentials (server-to-server), Device Code
- JWT (JSON Web Token): header.payload.signature — never store sensitive data in payload (base64-encoded, not encrypted); verify signature algorithm; reject `alg: none`; check `exp`, `iss`, `aud` claims
- Session management: secure + httpOnly + SameSite=Strict cookie attributes; rotate session ID on privilege escalation; absolute + idle timeout
- MFA: TOTP (RFC 6238) is standard; hardware keys (FIDO2/WebAuthn) are phishing-resistant; SMS is weak (SIM swap attacks)
- Zero-trust model: verify explicitly, use least privilege, assume breach — authenticate every request, do not trust network location

**Cryptography Essentials**
- Symmetric: AES-256-GCM (authenticated encryption — preferred over AES-CBC which requires separate MAC); ChaCha20-Poly1305 (better for mobile/low-power)
- Asymmetric: RSA-2048+ or RSA-4096 for key exchange; ECDSA P-256 or Ed25519 for signatures (faster, smaller keys)
- Hashing: SHA-256 / SHA-3 for integrity; NEVER MD5 or SHA-1 for security purposes
- Password hashing: bcrypt (cost factor ≥ 12), Argon2id (preferred — winner of Password Hashing Competition), scrypt — NEVER store plaintext or use fast hashes (SHA-256 for passwords is broken by GPU brute force)
- TLS: require TLS 1.2 minimum, prefer TLS 1.3 (forward secrecy by default); disable SSLv3, TLS 1.0, TLS 1.1; use HSTS
- Key management: key rotation schedules, HSMs for root keys, envelope encryption (data encrypted with data key; data key encrypted with KMS master key)

**Common Vulnerability Classes**
- SQL Injection: use parameterized queries / ORM — never string concatenation in SQL
- XSS (Cross-Site Scripting): stored, reflected, DOM-based — sanitize output (encode HTML entities); use CSP (Content Security Policy) headers; avoid `innerHTML`, `eval`, `document.write`
- CSRF: use CSRF tokens (synchronizer token pattern) or SameSite=Strict cookies; double-submit cookie pattern
- Path Traversal: validate and canonicalize file paths; reject `../` sequences; use allowlists not denylists
- Deserialization attacks: never deserialize untrusted data with native serializers; use safe alternatives (JSON over XML/Java serialization)
- Race conditions / TOCTOU: use database transactions with proper isolation levels; atomic file operations

**Infrastructure and Cloud Security**
- IAM least privilege: grant minimum permissions needed; use roles not long-lived credentials; rotate access keys
- Secret management: use Vault (HashiCorp), AWS Secrets Manager, GCP Secret Manager — never hardcode secrets; never commit secrets to git
- Network segmentation: VPC, security groups, NACLs; bastion hosts or VPN for admin access; no public-facing databases
- Container security: minimal base images (distroless, Alpine); run as non-root; read-only filesystem where possible; scan images for CVEs (Trivy, Snyk); no privileged containers
- Supply chain security: SBOM (Software Bill of Materials), dependency pinning, Sigstore/Cosign for artifact signing

**Incident Response**
- Phases: Preparation → Identification → Containment → Eradication → Recovery → Lessons Learned (NIST SP 800-61)
- Containment strategy: isolate affected systems; preserve evidence (forensic disk image before wipe); change all credentials; revoke tokens/sessions
- IOC (Indicators of Compromise) vs TTP (Tactics, Techniques, Procedures): IOCs change (IPs, hashes); TTPs are more stable — focus detection on TTPs
- Chain of custody for forensic evidence; legal hold obligations

### Decision Frameworks for the Knowledge Base

**Vulnerability Severity Assessment (CVSS v3):**
Base Score factors: Attack Vector (network/adjacent/local/physical), Attack Complexity (low/high), Privileges Required, User Interaction, Scope, Confidentiality/Integrity/Availability impact.
Critical: CVSS 9.0–10.0 → patch within 24–48 hours.
High: 7.0–8.9 → patch within 7 days.
Medium: 4.0–6.9 → patch within 30 days.
Low: 0.1–3.9 → schedule for next release cycle.

**Risk = Likelihood × Impact.** A CVSS-High vulnerability with no network exposure and authentication required may be lower actual risk than a CVSS-Medium with anonymous network access.

**Security vs Usability Tradeoffs:** Security controls that users work around are worse than weaker controls that are actually used. Design for the 99th percentile user, not the attacker (who will probe every path anyway).

---

## Domain 3: Machine Learning / AI Systems

### What the Expert Needs to Reason About

An ML Expert Advisor reasons about **model architecture selection, training stability, evaluation correctness, deployment reliability, and data quality**. It must distinguish between "this looks good in a notebook" and "this will behave correctly in production."

### Core Vocabulary (must be in the knowledge base)

**Model Architecture Decisions**
- Transformer architecture: attention mechanism (self-attention, cross-attention), positional encoding, layer normalization, feed-forward sublayer; encoder-only (BERT, RoBERTa — good for classification/embeddings), decoder-only (GPT family — good for generation), encoder-decoder (T5, BART — good for seq2seq tasks)
- CNN: translation invariance, pooling layers, receptive field; effective for spatial data (images, audio spectrograms, time series with local patterns)
- RNN/LSTM/GRU: sequential modeling, vanishing gradient problem (solved by LSTM gates), largely replaced by Transformers for NLP but still useful for edge inference
- Model selection heuristic: start with the simplest model that could work; add complexity only when simple models plateau

**Training Fundamentals**
- Loss functions: MSE/MAE for regression; Cross-Entropy for classification (with softmax); Binary Cross-Entropy for multi-label; CTC loss for sequence alignment (speech recognition); contrastive loss for embeddings (SimCLR, InfoNCE)
- Optimizers: Adam (adaptive learning rates — good default); AdamW (Adam + proper weight decay — preferred for transformers); SGD + momentum (slower convergence, often better generalization); Lion (newer, more token-efficient)
- Learning rate: most important hyperparameter; use cosine annealing with warmup for transformers; cyclical LR for CNNs; too high → divergence; too low → slow convergence or stuck in local minima
- Regularization: dropout (randomly zero activations during training — rate 0.1–0.5); weight decay (L2 penalty); data augmentation; early stopping; batch normalization (normalizes activations — speeds training, provides implicit regularization)
- Gradient issues: vanishing gradients → residual connections, layer norm, careful initialization (Kaiming/He for ReLU, Xavier/Glorot for sigmoid/tanh); exploding gradients → gradient clipping (clip_grad_norm, max_norm=1.0)

**Evaluation and Metrics**
- Classification: Accuracy (misleading for imbalanced classes); Precision (TP/(TP+FP)); Recall/Sensitivity (TP/(TP+FN)); F1 Score (harmonic mean of P and R); AUC-ROC (threshold-independent; 0.5 = random, 1.0 = perfect); Matthews Correlation Coefficient (MCC — best single metric for imbalanced binary classification)
- Regression: MAE (robust to outliers), MSE/RMSE (penalizes large errors more), R² (proportion of variance explained — can be negative if model is worse than predicting mean), MAPE (percentage error — undefined when target = 0)
- NLP generation: BLEU (n-gram overlap with reference — poor for creative tasks); ROUGE (recall-oriented overlap — used for summarization); BERTScore (semantic similarity using embeddings — better than n-gram metrics); human evaluation remains gold standard
- LLM-specific: perplexity (lower = better language model fit); hallucination rate; factual accuracy on benchmarks (MMLU, TruthfulQA, HellaSwag); instruction-following (MT-Bench, Alpaca Eval)

**Data Quality and Pipeline**
- Data leakage: future information in training data; label leakage (using target-correlated features that wouldn't be available at inference time); temporal leakage in time series (train on future, test on past)
- Train/validation/test split: temporal split for time series (never random); stratified split for imbalanced classes; test set must be held out until final evaluation — never tuned against
- Class imbalance: oversampling (SMOTE for tabular, augmentation for images); undersampling; class-weighted loss; threshold tuning at inference (don't use 0.5 as default)
- Feature engineering: normalization for distance-based models (KNN, SVM, linear); irrelevant for tree-based models (RF, XGBoost); missing values: imputation vs indicator features vs model-based imputation
- Dataset shift: covariate shift (input distribution changes); concept drift (relationship between input and output changes); label shift (output distribution changes); monitor with distribution tests (KL divergence, PSI — Population Stability Index)

**RAG (Retrieval-Augmented Generation)**
- Architecture: user query → embedding (dense retrieval) or keyword search (sparse/BM25) or hybrid → top-k chunks retrieved → context + query → LLM → response
- Chunking strategies: fixed-size (fast, misses semantic boundaries); sentence/paragraph (better); semantic chunking (cluster by meaning — best but expensive); parent-child chunking (retrieve small, pass large)
- Embedding models: `text-embedding-3-large` (OpenAI), `embed-english-v3.0` (Cohere), `bge-large` (BAAI — open source); cosine similarity for comparison; normalize vectors before HNSW indexing
- Vector databases: Pinecone (managed), Weaviate (self-hosted), Chroma (local/lightweight), Qdrant (self-hosted, Rust, fast), pgvector (Postgres extension — good for existing PG users)
- Reranking: first-pass retrieval by vector similarity; second-pass reranking by relevance (cross-encoder: slower but more accurate than bi-encoder); Cohere Rerank, FlashRank, cross-encoder models
- Evaluation: RAGAS framework (faithfulness, answer relevancy, context precision, context recall); manual spot-checking; hallucination detection

**MLOps and Production**
- Model serving: REST API (FastAPI + Uvicorn, TorchServe, TF Serving); batch inference (Ray, Spark); streaming inference (Kafka + Faust); serverless (Lambda, Cloud Functions — cold start penalty)
- Monitoring: prediction distribution drift (PSI threshold: 0.1 = minor, 0.2+ = significant); feature drift; latency (p50, p95, p99); error rate; model performance metrics on logged production samples
- A/B testing: shadow mode (log but don't serve), canary deployment (5–10% traffic), champion-challenger, proper statistical testing (min sample size, p-value < 0.05, practical significance threshold)
- Model registry: MLflow, W&B Model Registry, SageMaker Model Registry — version models, track lineage, promote stages (staging → production → archived)

---

## Domain 4: Legal / Compliance

### What the Expert Needs to Reason About

A Legal/Compliance Expert Advisor reasons about **regulatory obligations, risk exposure, and compliance requirements** that the project must satisfy. It does not give legal advice — it flags when human legal counsel is required and ensures the development team is asking the right questions.

### Core Vocabulary (must be in the knowledge base)

**Data Privacy Regulations**
- GDPR (EU/EEA): lawful basis for processing (consent, contract, legitimate interest, legal obligation, vital interest, public task); data subject rights (access, rectification, erasure/right to be forgotten, portability, objection, restriction of processing, not to be subject to automated decision-making); DPA (Data Processing Agreement) required for processors; breach notification: 72 hours to supervisory authority, without undue delay to data subjects if high risk; DPO (Data Protection Officer) required for large-scale processing of special categories
- CCPA/CPRA (California): right to know, delete, opt-out of sale/sharing, correct, limit use of sensitive PI; businesses with > $25M revenue or processing > 100k consumers; CPRA added right to opt-out of behavioral advertising; enforcement by California AG + California Privacy Protection Agency
- HIPAA (US healthcare): PHI (Protected Health Information) — 18 identifiers; Covered Entities + Business Associates must sign BAA; Minimum Necessary standard; Security Rule (administrative, physical, technical safeguards); Privacy Rule; Breach Notification Rule; civil penalties up to $1.9M per violation category per year
- Special categories (GDPR): health, biometric, genetic, racial/ethnic origin, political opinions, religious beliefs, trade union membership, sex life/orientation — require explicit consent or specific exceptions

**Security Compliance Frameworks**
- SOC 2 Type I vs Type II: Type I = point-in-time controls design; Type II = effectiveness over audit period (typically 6–12 months); Trust Service Criteria: Security (required), Availability, Processing Integrity, Confidentiality, Privacy
- ISO 27001: ISMS (Information Security Management System); risk-based approach; 93 controls in Annex A (ISO 27002); certification requires external audit
- PCI DSS (payment card): 12 requirements; SAQ vs full QSA assessment based on transaction volume; cardholder data environment (CDE) scoping is critical — minimize CDE scope through tokenization
- NIST CSF (Cybersecurity Framework): Identify → Protect → Detect → Respond → Recover; not mandatory but widely used as framework
- FedRAMP (US federal cloud): authorization levels Low/Moderate/High based on impact; requires continuous monitoring; ATO (Authority to Operate)

**Contract Law Essentials (for SaaS/API products)**
- SLA (Service Level Agreement): uptime commitments (99.9% = 8.7 hours downtime/year; 99.99% = 52 minutes/year); remedies (service credits, not liability); measurement methodology (how is downtime calculated?); exclusions (scheduled maintenance, force majeure, customer-caused)
- IP ownership: work-for-hire doctrine; assignment clauses; derivative works; open-source license obligations in commercial products
- Liability limitations: consequential damages waivers; liability caps (typically 1× or 12× monthly fees); indemnification provisions; IP indemnification
- Terms of Service / Privacy Policy: legally required statements vary by jurisdiction; dark patterns in consent flows create regulatory exposure

**Open-Source License Obligations**
- Copyleft (viral): GPL v2/v3 — derivative works must be open-sourced under GPL; AGPL — triggers on network use (SaaS); LGPL — allows linking without copyleft obligation
- Permissive: MIT, Apache 2.0 (patent grant + attribution), BSD (attribution) — can be used in proprietary products
- Creative Commons: CC0 (public domain), CC-BY (attribution), CC-BY-SA (share alike = viral), CC-NC (no commercial use) — primarily for content, not code
- SCA (Software Composition Analysis): track licenses of all dependencies; blocklist GPL/AGPL for commercial products without legal review; automate with FOSSA, Black Duck, Snyk License

---

## Domain 5: Blockchain / Web3

### What the Expert Needs to Reason About

A Blockchain Expert Advisor reasons about **smart contract security, protocol mechanics, gas optimization, and DeFi risks**. It must understand both the technical layer (EVM, opcodes, storage layout) and the economic layer (incentive design, MEV, liquidity mechanics).

### Core Vocabulary (must be in the knowledge base)

**Smart Contract Security**
- Reentrancy: attacker contract calls back into victim before state is updated; mitigate with checks-effects-interactions pattern (update state BEFORE external calls) or ReentrancyGuard (mutex)
- Integer overflow/underflow: Solidity < 0.8.0 had unchecked arithmetic; use SafeMath or Solidity 0.8+ (built-in overflow checks); be aware `unchecked` block bypasses this
- Access control: `onlyOwner` is not enough for complex systems; use OpenZeppelin AccessControl for role-based permissions; always emit events on permission changes
- Front-running / MEV (Maximal Extractable Value): bots monitor mempool, submit higher-gas transactions to front-run profitable opportunities; mitigations: commit-reveal schemes, private mempools (Flashbots Protect), slippage tolerance limits
- Oracle manipulation: price oracles (Chainlink, TWAP) can be manipulated in low-liquidity markets; use time-weighted averages (TWAP), not spot prices; use multiple oracle sources
- Flash loans: uncollateralized loans within one transaction; used for arbitrage AND attacks (price manipulation, governance attacks); mitigate by checking spot price vs TWAP divergence
- Common audit findings: missing access controls, uninitialized proxies, incorrect decimal handling (ERC-20 tokens use 18 decimals — arithmetic errors cause fund loss), unsafe external calls, missing slippage protection

**EVM Fundamentals**
- Storage layout: 32-byte slots; packed variables (multiple small variables in one slot saves gas); mappings don't pack; arrays store length in base slot; always use same Solidity version across upgradeable contract logic
- Gas optimization: storage writes (SSTORE) are expensive (20,000 gas for new slot); use memory for intermediate calculations; use calldata not memory for read-only function parameters; batch operations; use events instead of storage for historical data; minimize on-chain computation
- Proxy patterns: Transparent Proxy (admin/user selector clash handled by admin address check — OpenZeppelin standard), UUPS (upgrade logic in implementation — more gas efficient), Diamond (EIP-2535 — multiple facets)
- Events: cheap (log storage); indexed parameters allow filtering; cannot be read from contracts but can be read by off-chain

**DeFi Protocol Mechanics**
- AMM (Automated Market Maker): constant product formula x*y=k (Uniswap v2); concentrated liquidity (Uniswap v3 — LPs choose price range, higher capital efficiency but active management needed); impermanent loss = divergence loss from holding LP shares vs holding underlying assets
- Lending protocols: overcollateralized loans (Aave, Compound); liquidation mechanism (health factor, liquidation threshold, liquidation bonus incentivizes liquidators); interest rate models (utilization-based: low utilization = low rate, high utilization = high rate)
- Yield farming: liquidity mining (governance tokens as incentive), compounding strategies; understand APY vs APR; impermanent loss often exceeds yield farming rewards
- Governance: token-weighted voting; time-locks on execution (critical — prevents flash loan governance attacks); quorum requirements; vote delegation

---

## Domain 6: DevOps / Infrastructure

### What the Expert Needs to Reason About

A DevOps Expert Advisor reasons about **system reliability, deployment correctness, scaling tradeoffs, and infrastructure security**. It distinguishes between "works on my machine" and "survives production traffic at 3 AM."

### Core Vocabulary (must be in the knowledge base)

**Container and Orchestration**
- Kubernetes core primitives: Pod (smallest deployable unit, ephemeral), ReplicaSet (maintains desired replica count), Deployment (rolling updates, rollback), Service (stable network endpoint — ClusterIP/NodePort/LoadBalancer), ConfigMap + Secret (externalized config), PersistentVolumeClaim (storage)
- Health checks: liveness probe (is the container alive? restart if fails), readiness probe (is the container ready to receive traffic? remove from load balancer if fails), startup probe (give slow-starting apps time to initialize before liveness starts)
- Resource requests vs limits: requests = guaranteed allocation (scheduler uses this); limits = max allowed (CPU: throttled at limit; memory: OOMKilled at limit). Set both. Request/limit ratio > 2× is usually wrong.
- Horizontal Pod Autoscaler (HPA): scales based on metrics (CPU%, memory, custom); requires resource requests to be set; VPA (Vertical Pod Autoscaler) adjusts request/limits dynamically

**CI/CD Best Practices**
- Pipeline stages: lint → unit test → build → integration test → security scan → deploy to staging → smoke test → deploy to production
- Deployment strategies: rolling (zero-downtime, gradual — risk: in-flight requests during transition); blue-green (instant cutover, expensive — 2x resources); canary (route % of traffic to new version, monitor); feature flags (decouple deployment from release)
- GitOps: git as source of truth for cluster state; ArgoCD or Flux syncs cluster to git; changes via PRs — auditable, reversible
- Secrets in CI/CD: never put secrets in environment variables in CI YAML files committed to git; use CI/CD secret stores (GitHub Actions Secrets, HashiCorp Vault, AWS Secrets Manager with OIDC)

**Observability (The Three Pillars)**
- Metrics: time-series numeric data (Prometheus); USE method for resources (Utilization, Saturation, Errors); RED method for services (Rate, Errors, Duration/Latency); track p50/p95/p99 latency (average hides tail latency)
- Logs: structured logging (JSON — makes parsing reliable); log levels (DEBUG/INFO/WARN/ERROR — never log secrets, PII, or tokens); centralized logging (ELK stack, Loki, Datadog, Splunk); correlation IDs for distributed tracing
- Traces: distributed tracing follows a request across services (OpenTelemetry — vendor-neutral standard); span = unit of work; trace = collection of spans; parent-child span relationships; Jaeger, Zipkin, Tempo, Datadog APM
- SLI/SLO/SLA: SLI (what you measure: request success rate, latency p99); SLO (target: 99.9% success, p99 < 200ms); SLA (contractual commitment with penalties); error budget = 100% - SLO (spend it on velocity, protect it in production)

**Infrastructure as Code**
- Terraform: declarative, idempotent; plan → apply; state management (remote state in S3 + DynamoDB for locking); modules for reusability; `terraform destroy` is dangerous — protect with lifecycle rules and deletion protection
- Helm: Kubernetes package manager; chart = templated Kubernetes manifests; values.yaml for configuration; tiller was removed in v3 (security improvement); use Helm for complex applications, Kustomize for simpler overlays
- Idempotency: running IaC multiple times must produce the same result; avoid imperative steps in Terraform (use null_resource sparingly); use `create_before_destroy` lifecycle rule to avoid downtime

---

## Domain 7: Data Engineering

### What the Expert Needs to Reason About

A Data Engineering Expert Advisor reasons about **pipeline reliability, data quality, schema evolution, and query performance**. It distinguishes between one-time data loads and production pipelines that must run correctly every day at scale.

### Core Vocabulary (must be in the knowledge base)

**Pipeline Architecture Patterns**
- Batch vs streaming: batch (high throughput, latency minutes-to-hours, simpler); streaming (low latency, seconds, complex state management); lambda architecture (batch layer + speed layer — complex, often over-engineered); kappa architecture (streaming only, reprocess from event log)
- ELT vs ETL: ETL (transform before load — for compliance/PII scrubbing, limited warehouse storage); ELT (load raw, transform in warehouse — preferred for modern cloud DWH with abundant storage); EL (load only — simplest, defer all transformation)
- Orchestration: Airflow (DAGs, Python operators, mature ecosystem, complex); Prefect/Dagster (modern, better DX, native Python, observability built-in); dbt for SQL transformation layer; Kafka for event streaming
- Medallion architecture (Delta Lake pattern): Bronze (raw ingest, no transformation, append-only) → Silver (cleaned, deduplicated, conformed) → Gold (aggregated, business-ready, optimized for query patterns)

**Data Quality**
- Dimensions: completeness (no missing values in required fields), accuracy (matches source), consistency (same data in multiple places), timeliness (freshness SLAs), validity (conforms to schema/constraints), uniqueness (no duplicates)
- Frameworks: Great Expectations (Python, data contracts), dbt tests (schema tests, custom tests), Soda Core, Monte Carlo (observability platform)
- Schema enforcement: schema-on-write (strict — parquet, Delta Lake) vs schema-on-read (flexible — CSV in S3); evolution strategies: forward compatibility (new code reads old data), backward compatibility (old code reads new data), full compatibility
- Data lineage: track column-level lineage (which source columns produced which output columns); OpenLineage standard; integrate with Marquez, DataHub, or Atlan

**Query Performance**
- Partitioning: partition by high-cardinality columns used in WHERE filters (date, region, customer_id for large tables); avoid over-partitioning (many small files — use OPTIMIZE/VACUUM in Delta Lake)
- Clustering/Z-ordering: colocate related data on disk; Databricks ZORDER, Snowflake clustering keys; effective for range queries on non-partition columns
- File formats: Parquet (columnar, compressed, splittable — best for analytics); Delta Lake (Parquet + ACID transactions + DML); Iceberg (alternative to Delta, better table evolution); Avro (row-based, schema evolution, Kafka-native)
- Column pruning and predicate pushdown: query engines skip columns not in SELECT; push WHERE filters down to scan layer; critical for Parquet on S3 (avoid SELECT *)
- Vacuuming: Delta Lake retains old file versions for time travel; run VACUUM to remove old files and control storage costs; default retention 7 days — adjust based on needs

---

## Using This File: Builder's Checklist

When building an expert knowledge base from a domain above:

1. **Read the relevant domain section** — understand the structural framework and vocabulary
2. **Run Pass 1–5 of the research loop** — fill in depth beyond what this framework provides
3. **Check coverage** — the final knowledge base must include: terminology (defined precisely), decision frameworks (if X then Y), edge cases (where the standard approach fails), what to measure (specific metrics with interpretation guidance), and scope boundaries (what this expert does NOT advise on)
4. **Match or exceed the FINANCE_KNOWLEDGE.md bar** — if the user has provided their own domain knowledge file, treat it as the primary source and augment it, do not replace it
5. **Write numbered sections** — makes it easy for the agent to reference specific sections in its system prompt

> **Minimum length:** 300+ lines for working-knowledge. 500+ lines for practitioner-level. Shorter knowledge bases produce shallow agents.
