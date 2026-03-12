# AI/ML Cloud Service Attack Patterns

Attack patterns for AI/ML services deployed on major cloud platforms. These services are the fastest-growing cloud attack surface — enterprises deploying SageMaker, Vertex AI, Bedrock, and Azure OpenAI often misconfigure IAM, expose backing stores, or leave multi-API parity gaps.

**Boundary with ai-hunting:** This file covers IAM, network, storage, and configuration misconfigurations for AI cloud services. For model-behavior attacks (prompt injection, tool poisoning, memory poisoning, agent hijacking), use the `ai-hunting` skill.

---

## Table of Contents

- [API Parity Testing](#api-parity-testing)
- [User-Controlled Model Selection](#user-controlled-model-selection)
- [Backing-Store Exposure](#backing-store-exposure)
- [Execution Identity Overreach](#execution-identity-overreach)
- [Connector and Tool Abuse](#connector-and-tool-abuse)
- [AI-Adjacent Surfaces](#ai-adjacent-surfaces)
- [Data/Control Plane Separation](#datacontrol-plane-separation)
- [URL Validation Parser Differential](#url-validation-parser-differential)
- [Cloud AI Service Patterns by Provider](#cloud-ai-service-patterns-by-provider)

---

## API Parity Testing

Cloud AI platforms expose the same model through multiple API paths. Authorization and policy enforcement often differ between them — a restriction on one path may not exist on another.

**Why this matters:** Transport drift creates parity bugs. The 2026 AI cloud platforms increasingly expose OpenAI-compatible, native, streaming, and legacy API styles for the same models. Security policies (content filtering, rate limiting, auth) are often applied inconsistently.

| Provider | API Paths to Compare | What to Test |
|----------|---------------------|--------------|
| **AWS SageMaker** | `InvokeEndpoint` vs `InvokeEndpointWithResponseStream` vs `InvokeEndpointAsync` | Auth policy applied to sync but not async/streaming? Different IAM permissions per path? |
| **AWS Bedrock** | `InvokeModel` vs `Converse` vs `ConverseStream` vs Chat Completions (OpenAI-compatible) | Guardrails applied on `InvokeModel` but not `Converse`? Rate limits differ? Content filter bypass via streaming? |
| **GCP Vertex AI** | `Predict` vs `RawPredict` vs streaming vs OpenAI-compatible chat endpoint | `RawPredict` bypasses model-level validation? Streaming endpoint skips content filtering? |
| **Azure OpenAI** | Azure OpenAI API vs Foundry Agent Service vs legacy Assistants API vs On Your Data (deprecated, approaching retirement) | Auth required on main API but not Foundry? Legacy paths still active without current auth controls? |

**Test procedure:**
1. Identify all API paths the target uses to invoke AI models (check JS source, API docs, network traffic)
2. Send the same request to each path — compare auth requirements, content filtering, rate limiting
3. If one path has weaker controls, test for data exfiltration, content filter bypass, or elevated access
4. Check for legacy/deprecated API paths still active (especially Azure On Your Data, SageMaker v1 endpoints)

---

## User-Controlled Model Selection

Some AI platforms allow the caller to specify which model or variant to use. If the application passes user input to these selectors without validation, attackers can target different models with different capabilities or security postures.

| Provider | Selector | Risk |
|----------|----------|------|
| **AWS SageMaker** | `TargetModel` — selects model in multi-model endpoint | User swaps to a model with different permissions, output format, or capabilities |
| **AWS SageMaker** | `TargetVariant` — selects production variant (A/B testing) | Access variant with different model version, potentially without content filtering |
| **AWS SageMaker** | `SessionId` — stateful inference sessions | Session confusion — access another user's session state, context, or conversation |
| **AWS Bedrock** | Model ID in API call | Model downgrade — target a model without guardrails or with known jailbreak weaknesses |
| **GCP Vertex AI** | Endpoint ID + model deployment | Deployment version confusion — target older model version without safety tuning |
| **Azure OpenAI** | Deployment name in URL path | Deployment confusion — access a deployment with different content filters or system prompts |

**Test procedure:**
1. Intercept requests to AI service endpoints
2. Identify model/variant/deployment selectors in the request
3. Enumerate valid values (brute-force, error messages, API introspection)
4. Swap to alternative models/variants — test if auth, content filtering, or capabilities change
5. Test session IDs for IDOR — can you access another user's inference session?

---

## Backing-Store Exposure

The highest-value data in AI deployments isn't the model endpoint — it's the backing stores containing training data, prompts, embeddings, vector databases, fine-tuning datasets, and inference logs.

| Data Store | Where to Find | What It Contains | Impact |
|------------|---------------|------------------|--------|
| **S3/GCS/Blob buckets** | Model artifact storage, training data, batch output | Source code, PII in training data, model weights | Data breach, model theft |
| **Vector stores** | Pinecone, Weaviate, Chroma, OpenSearch, Azure AI Search | Embeddings of private documents, RAG knowledge base | Indirect data exfiltration |
| **Prompt/trace logs** | CloudWatch, Cloud Logging, Azure Monitor, dedicated log buckets | User prompts, model responses, system prompts | PII exposure, system prompt leak |
| **Fine-tuning datasets** | S3/GCS training buckets, SageMaker data channels | Curated data with potential PII, proprietary info | Data breach |
| **Batch output buckets** | S3 output paths for batch inference | Inference results on bulk data | Mass data exposure |
| **Cosmos DB / DynamoDB** | Azure Foundry agent state, conversation history | Full conversation logs, tool call results | PII, internal data exposure |
| **Agent action schemas** | S3 (Bedrock), GCS (Vertex), Azure Storage | API schemas defining what actions agents can take | Discover internal APIs, find injection points |

**Test procedure:**
1. Find AI service configurations (CloudFormation, Terraform, ARM templates, console screenshots)
2. Identify backing-store references (bucket names, connection strings, endpoint URLs)
3. Test each backing store for public access or overly permissive IAM
4. Check if logs contain sensitive user data (prompts are often PII-rich)
5. Vector stores are especially valuable — a single query can return chunks of private documents

---

## Execution Identity Overreach

AI services run with IAM roles/service accounts that often have excessive permissions. The execution identity for a model endpoint or AI agent frequently has access to secrets, storage, or admin APIs far beyond what's needed.

| Provider | Service | Identity to Check | Common Overreach |
|----------|---------|-------------------|------------------|
| **AWS** | SageMaker endpoints | Execution role | Read access to all S3 buckets, Secrets Manager, other services |
| **AWS** | Bedrock Agents | Agent execution role | `bedrock:InvokeModel` + `s3:*` + `secretsmanager:GetSecretValue` |
| **AWS** | Bedrock Knowledge Bases | KB role | Read access to entire S3 prefix, not just the KB bucket |
| **GCP** | Vertex AI endpoints | Service account | `storage.objectViewer` on all project buckets, compute admin |
| **GCP** | Vertex AI RAG Engine | Service account | Access to Cloud Storage, Drive, Slack, Jira, SharePoint connectors |
| **Azure** | Azure OpenAI | Managed identity | `Cognitive Services OpenAI Contributor` + storage/key vault access |
| **Azure** | Foundry Agent Service | Managed identity | BYO Cosmos DB, Storage, AI Search — identity can read all connected stores |

**Test procedure:**
1. If you have SSRF to metadata → extract the role/SA credentials
2. Enumerate what the identity can access: `aws sts get-caller-identity` → `aws iam list-attached-role-policies`
3. Check if the AI service identity can access secrets, other services, or admin APIs
4. Bedrock Agents: check if the action-group schema grants API access beyond the agent's intended scope
5. Look for overprivileged identities that can pivot to other cloud services

---

## Connector and Tool Abuse

Modern AI platforms connect to external data sources and tools. These connectors often have overly broad permissions.

### AWS Bedrock

| Component | Attack Surface |
|-----------|---------------|
| Knowledge Bases | S3 data source bucket permissions — can the KB role read beyond its intended prefix? |
| Action Groups | API schemas stored in S3 — can you read/modify the schema to change agent behavior? |
| Guardrails | Applied per-model; test if bypassable via `Converse` vs `InvokeModel` |

### GCP Vertex AI

| Component | Attack Surface |
|-----------|---------------|
| RAG Engine data sources | Cloud Storage, Drive, Slack, Jira, SharePoint — test connector permissions |
| Extensions | Custom tool definitions — can untrusted input reach extension APIs? |
| Reasoning Engine | Custom Python functions deployed as endpoints — test for code injection |

### Azure Foundry Agent Service

| Component | Attack Surface |
|-----------|---------------|
| BYO resources | Cosmos DB, Azure Storage, AI Search — test access control on each |
| File Search tool | Azure Blob Storage — is the file search corpus overly broad? |
| Code Interpreter | Sandboxed compute — test for escape, network access, file system access |

---

## AI-Adjacent Surfaces

Don't just test the inference endpoint — test the management plane, notebooks, and development surfaces:

| Surface | Provider | Test |
|---------|----------|------|
| **SageMaker Studio / Notebooks** | AWS | Jupyter notebook instances run as root by default. Check for exposed notebook URLs, presigned URLs, IAM role escalation |
| **Multi-model endpoints** | AWS | `TargetModel` parameter — can you switch to a different model? Test for model IDOR |
| **Vertex AI Workbench** | GCP | Notebook instances with service account credentials. Check for public access, SA overreach |
| **Vertex AI Pipelines** | GCP | Pipeline definitions with embedded credentials. Check for injection in pipeline params |
| **Azure ML Studio** | Azure | Workspace access, compute instance exposure, registered model access |
| **Azure Foundry Portal** | Azure | Agent configuration access, tool definitions, conversation history |
| **Model registries** | All | Can you list, download, or modify registered models? Model weights are high-value targets |
| **Training jobs** | All | Job definitions may contain hyperparameters with embedded secrets, or write to overprivileged output locations |

---

## Data/Control Plane Separation

For AI cloud services, clearly separate findings into data plane (can invoke model, get predictions) vs. control plane (can read artifacts, logs, prompts, manage infrastructure):

| Plane | Examples | Severity |
|-------|----------|----------|
| **Data plane only** | Can invoke model endpoint, get predictions with own data | Low-Medium (self-impact) |
| **Data plane cross-user** | Can access another user's inference session, conversation history | High-Critical |
| **Control plane read** | Can read training data, prompts, embeddings, model artifacts | High-Critical (data breach) |
| **Control plane write** | Can modify model, fine-tuning data, guardrails, agent schemas | Critical (model poisoning, behavior modification) |
| **Identity plane** | Can assume AI service role, access other cloud services via overreach | Critical (lateral movement) |

**Reporting tip:** Always frame AI cloud findings in terms of which plane was compromised and what data/capability was accessed. "I could invoke the SageMaker endpoint" (data plane, self-impact) is very different from "I could read the training data bucket via the endpoint's execution role" (control plane + identity, high-critical).

---

## Worked Example: SageMaker Misconfiguration Chain

**Scenario:** Bug bounty target uses SageMaker for a customer-facing AI feature.

1. **Recon:** JS source reveals SageMaker endpoint URL and `InvokeEndpoint` API calls
2. **API parity test:** Target calls `InvokeEndpoint` with auth, but `InvokeEndpointWithResponseStream` doesn't check auth
3. **Session confusion:** `SessionId` parameter in streaming endpoint — enumerate sessions, access another user's conversation
4. **Execution role test:** SSRF on target reaches metadata, returns SageMaker execution role credentials
5. **Role overreach:** Execution role has `s3:GetObject` on `*` — can read all S3 buckets including training data with PII
6. **Impact:** Unauthenticated streaming → session IDOR → SSRF → credential extraction → training data with 50K customer records

**Severity:** Critical (CVSS 9.8). Three separate findings: auth bypass on streaming, session IDOR, IAM overreach. Report as chain with individual severity per component.

---

## URL Validation Parser Differential

AI/ML services frequently use one library to validate URLs (e.g., block private IPs for SSRF protection) and a different library to make the actual HTTP request. Discrepancies between the two parsers create SSRF bypasses that evade server-side protections.

**Why this matters:** This is a candidate vulnerability class to test whenever a service validates URLs before fetching. Using different libraries for validation and execution is a risk factor — a vulnerability exists when the two parsers disagree on attacker-controlled URL components (host, authority, path).

### CVE-2026-25960: vLLM MediaConnector Parser Differential (CVSS 7.1 NVD / 5.4 GitHub — scores dispute as of March 2026)

**Affected:** vLLM >= 0.15.1, < 0.17.0 (AI inference engine, widely deployed for enterprise LLM serving)

**Root cause:** vLLM's SSRF fix (for CVE-2026-24779) validates URLs with `urllib3.util.parse_url()` but makes HTTP requests via `aiohttp` (which uses `yarl` internally). The specific differential is backslash-`@` parsing: `urllib3` treats backslash as a path separator (extracting a benign-looking host), while `yarl` treats it differently, allowing the actual HTTP request to reach a different host than the one validated.

**Attack path:**
1. User submits crafted URL to `load_from_url_async` (image/media in LLM prompt)
2. `urllib3` parses URL → extracts hostname that appears allowed
3. `aiohttp`/`yarl` parses same URL → resolves to a different host (potentially internal services)
4. Attacker gains read access to internal HTTP endpoints reachable from the vLLM deployment

**Fixed:** v0.17.0 (URL normalized before network execution)

### Generalized Test Procedure: Parser Differential SSRF

This pattern applies to any AI/ML service (or any web service) that validates URLs before fetching content:

| Step | Action | What to look for |
|------|--------|-----------------|
| 1 | Identify URL input points | Image/media URLs in prompts, webhook URLs, callback URLs, connector config URLs, file import URLs |
| 2 | Determine validation library | Check error messages, response timing, or source code for which URL parser is used for validation |
| 3 | Determine execution library | Which HTTP client makes the actual request? (requests, aiohttp, httpx, urllib3, curl) |
| 4 | Test parser-specific bypass payloads | See payload table below |
| 5 | Verify internal access | Did the request reach `169.254.169.254`, `localhost`, or internal services? |

**Documented bypass payload (CVE-2026-25960):**

The specific differential exploited in vLLM is backslash-`@` handling: `urllib3.util.parse_url` treats `\` as a path separator, while `yarl` (used by `aiohttp`) treats it differently, causing host extraction to disagree. Example: a URL like `http://allowed-host\@internal-host/path` passes validation (urllib3 sees `allowed-host`) but the HTTP request goes to `internal-host`.

**Generic SSRF normalization payloads** (not parser-pair-specific — test all of these against any SSRF filter):

| Payload | Technique | Why Filters Miss It |
|---------|-----------|-------------------|
| `http://[::ffff:169.254.169.254]/` | IPv6-mapped IPv4 | Filters checking decimal only |
| `http://0xA9.0xFE.0xA9.0xFE/` | Hex octets | Filters expecting dotted-quad decimal |
| `http://2852039166/` | Decimal IP notation | Filters expecting dotted-quad |
| `http://169.254.169.254\t.public.com/` | Tab in hostname | Some parsers treat as part of host, others split |
| `http://169.254.169.254:80@public.com/` | Port/userinfo ambiguity | Parser disagrees on which part is host vs userinfo |
| `http://public.com@169.254.169.254/` | Userinfo prefix | Some validators extract userinfo as host |

**Known CVE-backed vulnerable library pairs:**

| Validation Library | Execution Library | Documented Differential | CVE |
|-------------------|-------------------|------------------------|-----|
| `urllib3.util.parse_url` | `aiohttp` (yarl) | Backslash-`@` host/userinfo parsing | CVE-2026-25960 |

**Potential anti-patterns** (theoretical risk — not backed by specific CVEs, but worth testing when identified in targets):

| Validation Library | Execution Library | Potential Differential |
|-------------------|-------------------|-----------------------|
| `urllib.parse.urlparse` | `requests` | Backslash handling, authority extraction |
| Custom regex | `httpx` / `aiohttp` | Incomplete IP format or encoding coverage |
| `java.net.URL` | `java.net.URI` → HttpClient | Authority parsing, DNS resolution differences |

**Where to hunt:** Any service accepting user-provided URLs where you can identify that validation and execution use different URL parsing libraries. AI/ML services are particularly prone because they frequently accept media URLs in prompts. Also applies to: webhook handlers, OAuth redirect validation, image proxy services, PDF/document import, any "fetch from URL" feature.

**Stop conditions:** If server returns the same error for all bypass variants and you can't determine the validation/execution library pair, move on. Different libraries are a risk factor, not a guarantee of vulnerability — a vuln exists only when the two parsers disagree on attacker-controlled URL components.

---

## Quick Reference: Cloud AI Endpoints to Test

| Provider | Service | Endpoint Pattern | Auth Check |
|----------|---------|-----------------|------------|
| AWS | SageMaker | `https://runtime.sagemaker.{region}.amazonaws.com/endpoints/{name}/invocations` | SigV4 |
| AWS | Bedrock | `https://bedrock-runtime.{region}.amazonaws.com/model/{id}/invoke` | SigV4 |
| GCP | Vertex AI | `https://{region}-aiplatform.googleapis.com/v1/projects/{proj}/locations/{loc}/endpoints/{id}:predict` | Bearer token |
| Azure | OpenAI | `https://{resource}.openai.azure.com/openai/deployments/{deploy}/chat/completions` | API key or Entra token |
| Azure | Foundry | `https://{resource}.services.ai.azure.com/agents/v1.0/assistants` | Entra token |
