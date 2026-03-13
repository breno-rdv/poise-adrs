# 0003. AI Pipeline for Vehicle Advertising

Date: 2026-03-12

## Status

Proposed

## Context

ADR 0001 established a microservices architecture for the Poise platform, and ADR 0002 identified the Advertising Service as the bounded context responsible for transforming dealer-provided vehicle media into a publishable listing.

Dealers will submit a video describing a car they want to advertise. The platform must turn that raw media into structured vehicle metadata, curated images, and usable ad copy without forcing the dealer to manually enter every detail. This workflow has several constraints:

- Video processing, transcription, image extraction, and generative AI calls are computationally expensive and have variable latency.
- The system must remain responsive at the API layer even when AI providers, media processing, or downstream storage are slow.
- AI outputs are probabilistic, so the platform must handle low-confidence results, retries, and reprocessing explicitly.
- Extracted media and text may contain sensitive or regulated content such as license plates, personal details, or dealership-specific notes.
- The pipeline must produce outputs that other services can consume asynchronously, including vehicle metadata and generated listing assets.

## Decision

We will implement the AI advertising workflow as an **event-driven pipeline** within the Advertising Service, with durable media storage, asynchronous processing stages, and explicit output events for downstream consumers.

### System Design

The pipeline will follow these stages:

1. The dealer submits a video through the platform API.
2. The API persists the original media in object storage and enqueues a processing request.
3. A media extraction worker produces:
   - audio for transcription
   - candidate frames for vehicle imagery
   - normalized technical metadata required by later stages
4. Specialized AI steps generate:
   - transcript-derived structured facts
   - vision-derived vehicle metadata and feature detection
   - listing descriptions and tags from the combined prompt context
5. A publication step stores the approved artifacts and emits domain events such as `CarMetadataCreated` and `CarDataCreated`.
6. All the steps will be coordinated using a a hibrid approach, using orchestration and Ledger pattern.

### Architectural rules

- **Asynchronous by default:** The API request ends after ingestion is acknowledged. All heavy processing happens out of band.
- **Stage isolation:** Media extraction, transcription, metadata generation, description generation, and publication remain separate stages so they can evolve, retry, and scale independently.
- **Durable handoff between stages:** Every stage reads from and writes to durable storage or queues rather than depending on in-memory chaining.
- **Event contracts:** Outputs are published as explicit domain events so catalog, search, and listing-management capabilities can react without tight coupling.
- **Managed AI services behind adapters:** Cloud-native services such as transcription and foundation-model inference may be used, but they will be called through service-owned adapters so the domain flow is not hard-wired to a single vendor.

### Required operating safeguards

- **Idempotency:** Each processing request must carry a stable media/job identifier so retries do not create duplicate assets or duplicate downstream events.
- **Confidence gating:** Metadata or generated copy below an agreed confidence threshold must be routed to human review instead of being auto-published.
- **Privacy controls:** The pipeline must support redaction or suppression of sensitive extracted content, especially license plates, personal information, and incidental audio.
- **Prompt and model versioning:** Generated outputs must record which prompt template and model version produced them to support audits and controlled reprocessing.
- **Observability:** Each stage must emit traceable job status, latency, failure reasons, retry counts, and quality metrics.
- **Failure handling:** Poison messages, provider outages, and malformed media must transition to explicit failed states and dead-letter flows instead of silent drops.

## Consequences

### Positive

- Keeps the dealer-facing API responsive even when AI and media-processing steps take several seconds or minutes.
- Allows each stage to scale independently according to workload and cost profile.
- Reduces coupling between ingestion, extraction, enrichment, and publication concerns.
- Makes reprocessing possible when prompts, models, or extraction heuristics improve.
- Creates clear integration points for downstream services through domain events instead of synchronous dependencies.
- Introduces operational guardrails for probabilistic AI behavior, including confidence thresholds and review paths.

### Negative

- Increases architectural and operational complexity compared with a synchronous single-service flow.
- Requires careful event design, idempotency handling, and state tracking across multiple stages.
- Introduces eventual consistency; listings and metadata will not be immediately available after upload.
- Adds cloud cost sensitivity because storage, queueing, media processing, transcription, and model inference all scale with usage.
- Requires explicit governance for model quality, prompt changes, and privacy compliance.

### Neutral

- AWS services shown in the current diagram are treated as the initial implementation direction, but the long-term architectural commitment is to the pipeline pattern and contracts, not to individual vendor APIs.
- Some vehicles may still require manual dealer correction or reviewer approval before publication; this is a normal part of the operating model, not an exception.
- Multiple output artifacts will exist for a single upload, including original media, derived images, transcript text, metadata, and generated copy.

## Alternatives Considered

### Synchronous request-response AI processing

**Rejected.** Running transcription, frame extraction, metadata generation, and description generation inside the initial API request would increase tail latency, reduce reliability, and make client experience dependent on the slowest provider call. It would also make retries and partial recovery far more difficult.

### Single-step generative AI workflow

**Rejected.** Sending the raw video or loosely preprocessed outputs into one large prompt would simplify the diagram, but it would reduce transparency, make failures harder to localize, and prevent targeted quality improvements by stage. Separate extraction and enrichment stages give us better control over cost, explainability, and reprocessing.

### Manual or semi-manual listing creation by dealers

**Rejected.** This would lower system complexity, but it would undermine the product goal of accelerating inventory creation and standardizing listing quality across dealers. Manual data entry can remain as a fallback or correction path, not as the primary flow.

### Custom in-house ML stack for every step

**Rejected for now.** Training and operating custom speech, vision, and generation models would create substantial platform overhead before product-market fit is proven. Managed AI services and foundation models offer faster iteration, provided we keep provider dependencies behind adapters.

## References

- [0001. Overall Architecture for Poise Platform](./0001-overall-architecture.md)
- [0002. Domain-Driven Design and Service Boundaries](./0002-domain-driven-design-service-boundaries.md)
- `resources/ai-pipeline.drawio`
