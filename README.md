# Multimodal Customer Support Simulator

A local customer-support simulation that checks text, image, video, and audio inputs before they reach a Gemini-powered customer agent. Unsafe content is blocked with structured feedback, while OpenTelemetry traces make the full moderation path inspectable in Arize Phoenix.

This repository packages a completed and extended implementation of an existing reference architecture as a reproducible portfolio prototype. It demonstrates multimodal safety gating, typed LLM outputs, API integration, evaluation, and observability; it is not presented as a production safety system or as an entirely from-scratch architecture.

## System workflow

1. A support representative sends text or media through the Gradio interface.
2. Gradio routes each item to the matching authenticated FastAPI endpoint.
3. A modality-specific Pydantic AI agent calls Gemini and returns a typed moderation result.
4. Flagged content is blocked and the rationale is shown to the representative.
5. Safe content reaches the simulated customer agent so the conversation can continue.
6. Conversation, turn, moderation, and model spans are exported to Phoenix.

```mermaid
flowchart LR
    user["Support representative"] --> ui["Gradio multimodal UI"]
    ui --> api["FastAPI moderation API"]
    api --> agents["Text / image / video / audio agents"]
    agents --> model["Google Gemini"]
    model --> result["Typed Pydantic result"]
    result --> gate{"Any policy flag?"}
    gate -->|Yes| blocked["Block and explain"]
    gate -->|No| customer["Simulated customer agent"]
    customer --> ui
    ui -. traces .-> phoenix["OpenTelemetry + Phoenix"]
```

## What the implementation demonstrates

- Structured moderation contracts for text, image, video, and audio, including an audio transcription.
- A single `is_flagged` decision contract with safe `False` defaults for every moderation flag.
- Bearer-authenticated FastAPI endpoints and a multimodal Gradio client.
- Blocking logic that prevents flagged content from reaching the customer agent.
- Conversation-level and turn-level spans, including moderation feedback attributes.
- Rule-based and LLM-as-a-judge evaluation suites for each modality.
- A locked Python 3.12 environment and CI checks for tests and formatting.

## Contribution boundary

The portfolio work completes and extends a supplied project scaffold. The distinction matters because repository evidence should be interview-defensible.

| Completed or extended in this portfolio implementation | Retained from the reference scaffold |
| --- | --- |
| Text and image moderation-agent execution paths | Core FastAPI endpoint structure and static bearer-token pattern |
| Typed audio result schema, Boolean defaults, and `is_flagged` behavior across modalities | Audio, video, and simulated-customer agent foundations |
| Multimodal Gradio routing, blocking behavior, message history, and trace hierarchy | Base environment, tracing, evaluation, and test structure |
| Text and image evaluation cases, validation artifacts, documentation, and CI | Fictional ACME scenario and supplied sample media |

See [`NOTICE.md`](NOTICE.md) for the upstream reference and licensing boundary.

## Stack

| Layer | Technology |
| --- | --- |
| Model integration | Pydantic AI, Google Gemini |
| Contracts | Pydantic |
| API | FastAPI, Uvicorn |
| Interface | Gradio |
| Observability | OpenTelemetry, OpenInference, Arize Phoenix |
| Evaluation | Pydantic Evals, deterministic assertions, LLM-as-a-judge |
| Reproducibility | Python 3.12, uv, `uv.lock` |

## Run locally

### Requirements

- Python 3.12
- [`uv`](https://docs.astral.sh/uv/)
- A Google AI Studio Gemini API key

Install the locked environment:

```bash
uv sync --locked
```

Create a private environment file and replace the placeholder values:

```bash
cp env.example .env
```

```dotenv
USER_API_KEY=replace-with-a-local-bearer-token
GEMINI_API_KEY=replace-with-your-google-ai-studio-key
DEFAULT_GOOGLE_MODEL=gemini-2.5-flash-lite
```

Start Phoenix, FastAPI, and Gradio together:

```bash
uv run multimodal-moderation
```

| Service | Local URL |
| --- | --- |
| Gradio application | [http://localhost:7860](http://localhost:7860) |
| FastAPI documentation | [http://localhost:8000/docs](http://localhost:8000/docs) |
| Phoenix traces | [http://localhost:6006/projects](http://localhost:6006/projects) |

## API surface

All endpoints use `Authorization: Bearer <USER_API_KEY>`.

| Method | Endpoint | Input |
| --- | --- | --- |
| `POST` | `/api/v1/moderate_text` | JSON body with a `text` field |
| `POST` | `/api/v1/moderate_image_file` | Multipart upload named `file` |
| `POST` | `/api/v1/moderate_video_file` | Multipart upload named `file` |
| `POST` | `/api/v1/moderate_audio_file` | Multipart upload named `file` |
| `GET` | `/api/v1/health` | No body; authentication is still required |

Example:

```bash
curl -X POST http://localhost:8000/api/v1/moderate_text \
  -H "Authorization: Bearer replace-with-your-user-api-key" \
  -H "Content-Type: application/json" \
  -d '{"text":"Welcome to support. How can I help?"}'
```

## Validation

CI uses the locked dependency graph and runs formatting checks plus the complete non-integration test suite:

```bash
uv lock --check
uv sync --locked
uv run black --check multimodal_moderation evals tests
uv run isort --check-only multimodal_moderation evals tests
uv run pytest tests -m "not integration" -vv
```

The optional integration test makes a live Gemini call and consumes API quota:

```bash
uv run pytest tests -m integration -vv
```

Historical execution records and screenshots are collected in [`docs/validation`](docs/validation/README.md). The saved evaluation run used one repeat over a small set of cases, so its percentages are regression evidence rather than a statistically robust benchmark.

Run the modality evaluations with:

```bash
uv run evals/text/test_cases.py
uv run evals/image/test_cases.py
uv run evals/audio/test_cases.py
uv run evals/video/test_cases.py
```

## Important limitations

- Model-based moderation is probabilistic and can produce false positives or false negatives.
- The static bearer token is suitable only for a local prototype, not a production identity boundary.
- Gradio limits individual uploads to 5 MB, but the FastAPI upload endpoints do not independently enforce that limit.
- Trace data can include user text, and media is copied to `uploaded_media/` for local visualization. Do not use real customer data, secrets, or PII.
- The stored evaluation set contains only three text cases, three image cases, two audio cases, and two video cases.
- Major dependency upgrades require a compatibility pass across Pydantic AI, Gradio, Phoenix, and Pydantic Evals.

## Provenance and license

The fictional scenario and initial scaffold were supplied in Udacity's public [`cd13331-multimodal-public`](https://github.com/udacity/cd13331-multimodal-public) repository. The completed and extended scope is documented above and in [`NOTICE.md`](NOTICE.md). Starter-derived material remains subject to the upstream educational-content terms in [`LICENSE.md`](LICENSE.md); this combined repository does not assert a blanket MIT license.
