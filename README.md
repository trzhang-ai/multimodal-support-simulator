# OmniTrainer — Multimodal Customer Service Trainer

OmniTrainer is an AI-powered customer-service training application for the fictional **ACME Enterprise**. A trainee acts as a support representative while a Google Gemini-powered agent plays an upset customer whose ACME Power Widget Pro has stopped working.

Before a trainee's message or uploaded media reaches the simulated customer, specialized moderation agents inspect it for safety and professionalism. Flagged content is blocked and accompanied by an explanation so the trainee can improve the response.

> **Project status:** Completed learning project based on provided starter scaffolding. It is a local proof of concept, not a production moderation or customer-service system.

## Features

- Moderates text, images, video, and audio with specialized Pydantic AI agents.
- Detects personally identifiable information (PII), unfriendly or unprofessional language, disturbing visual content, and low-quality media.
- Transcribes audio as part of the audio moderation result.
- Returns typed, structured Pydantic results with moderation flags and a rationale.
- Blocks flagged trainee content before it reaches the simulated customer.
- Provides a multimodal Gradio chat interface with moderation feedback.
- Exposes the moderation agents through authenticated FastAPI endpoints.
- Records conversation, moderation, and LLM spans with OpenTelemetry and Arize Phoenix.
- Includes unit/integration tests and LLM-based evaluation suites for every modality.

## How it works

1. The trainee sends text or media through the Gradio interface.
2. Gradio forwards each item to the appropriate FastAPI moderation endpoint.
3. A specialized moderation agent calls Gemini and returns a typed Pydantic result.
4. If any configured safety flag is `true`, the content is blocked and the rationale is shown to the trainee.
5. If all content is safe, it is sent to the Gemini-powered customer agent and the conversation continues.
6. OpenTelemetry spans make the conversation flow and timings visible in Phoenix.

```mermaid
flowchart LR
    trainee["Trainee support agent"] --> gradio["Gradio chat UI<br/>localhost:7860"]
    gradio --> api["FastAPI moderation service<br/>localhost:8000"]
    api --> agents["Text / Image / Video / Audio agents"]
    agents --> gemini["Google Gemini"]
    gemini --> schemas["Pydantic moderation results"]
    schemas --> decision{"Any unsafe flag?"}
    decision -->|Yes| blocked["Block content and show feedback"]
    decision -->|No| customer["Gemini-powered customer agent"]
    customer --> gradio
    gradio -. OpenTelemetry traces .-> phoenix["Arize Phoenix<br/>localhost:6006"]
    customer -. Instrumented LLM spans .-> phoenix
```

## Moderation outputs

| Modality | Structured checks |
| --- | --- |
| Text | `contains_pii`, `is_unfriendly`, `is_unprofessional`, `rationale` |
| Image | `contains_pii`, `is_disturbing`, `is_low_quality`, `rationale` |
| Video | `contains_pii`, `is_disturbing`, `is_low_quality`, `rationale` |
| Audio | `transcription`, `contains_pii`, `is_unfriendly`, `is_unprofessional`, `rationale` |

## Architecture and technology

| Component | Responsibility | Technology |
| --- | --- | --- |
| Moderation agents | Modality-specific prompts and model calls | Pydantic AI, Google Gemini |
| Output schemas | Typed moderation flags and rationales | Pydantic |
| Customer agent | Simulates the ACME customer across multiple chat turns | Pydantic AI, Google Gemini |
| Backend | Authenticated moderation endpoints | FastAPI, Uvicorn |
| Frontend | Multimodal training conversation and feedback | Gradio |
| Observability | Conversation hierarchy, timing, attributes, and model spans | OpenTelemetry, Arize Phoenix, OpenInference |
| Evaluations | Rule-based and LLM-as-a-judge quality checks | Pydantic Evals |

## Repository structure

```text
.
├── multimodal_moderation/
│   ├── agents/                 # Text, image, video, audio, and customer agents
│   ├── types/                  # Pydantic result and model-choice types
│   ├── app.py                  # Starts Phoenix, FastAPI, and Gradio together
│   ├── fastapi_app.py          # REST moderation API
│   ├── gradio_app.py           # Multimodal training interface
│   ├── tracing.py              # OpenTelemetry/Phoenix setup
│   ├── env.py                  # Environment and model configuration
│   └── utils.py                # File-type detection
├── evals/                      # Text, image, video, and audio evaluation suites
├── tests/                      # Automated tests and test media
├── env.example                 # Environment-variable template
├── pyproject.toml              # Package metadata and dependencies
└── uv.lock                     # Reproducible dependency lockfile
```

## Getting started

### Prerequisites

- Python 3.12
- [`uv`](https://docs.astral.sh/uv/getting-started/installation/)
- A Gemini API key from Google AI Studio, or the course-provided Vocareum credentials

### 1. Install the locked environment

From the repository root:

```bash
uv sync --locked
```

This creates the project-local `.venv` and installs the versions recorded in `uv.lock`. If your editor asks for an interpreter, select:

```text
.venv/bin/python
```

### 2. Configure environment variables

Create a private `.env` file from the included template:

```bash
cp env.example .env
```

Configure the three required variables:

```dotenv
USER_API_KEY=replace-with-a-local-api-token
GEMINI_API_KEY=replace-with-your-gemini-api-key
DEFAULT_GOOGLE_MODEL=gemini-2.5-flash-lite
```

`USER_API_KEY` protects the local FastAPI endpoints and only needs to be a non-empty value for this demo. Do not commit `.env`; it is already ignored by Git.

If you are using a regular Google AI Studio key, remove or comment out the Vocareum-only line from the copied `.env`:

```dotenv
# GOOGLE_GEMINI_BASE_URL=https://gemini.vocareum.com
```

Optional variables and their defaults:

| Variable | Default | Purpose |
| --- | --- | --- |
| `EVAL_JUDGE_MODEL` | `DEFAULT_GOOGLE_MODEL` | Model used by LLM-as-a-judge evaluators |
| `EVAL_NUM_REPEATS` | `1` | Number of times each eval case is repeated |
| `API_BASE_URL` | `http://localhost:8000` | FastAPI base URL used by Gradio |
| `PHOENIX_URL` | `http://127.0.0.1:6006` | Phoenix endpoint used by the trace exporter |

## Run the application

Start Phoenix, FastAPI, and Gradio together:

```bash
uv run multimodal-moderation
```

Wait for all three services to start, then open:

| Service | URL |
| --- | --- |
| Training application | [http://localhost:7860](http://localhost:7860) |
| FastAPI documentation | [http://localhost:8000/docs](http://localhost:8000/docs) |
| Phoenix traces | [http://localhost:6006/projects](http://localhost:6006/projects) |

Press `Ctrl+C` in the terminal to stop the application.

### Suggested training flow

1. Greet the customer and ask how you can help.
2. Respond to the complaint about the non-working ACME Power Widget Pro.
3. Try text, image, video, or audio input and review the moderation feedback panel.
4. Deliberately try an unfriendly message to see it blocked before reaching the customer agent.
5. Continue with a professional response and attempt to resolve the issue.
6. Select **End Conversation** so the conversation span is closed in Phoenix.

Gradio limits uploaded media to 5 MB per file. Unsupported file types are rejected before they are sent to the model.

## REST API

All endpoints use bearer authentication with the value configured as `USER_API_KEY`.

| Method | Endpoint | Input |
| --- | --- | --- |
| `POST` | `/api/v1/moderate_text` | JSON body with a `text` field |
| `POST` | `/api/v1/moderate_image_file` | Multipart upload named `file` |
| `POST` | `/api/v1/moderate_video_file` | Multipart upload named `file` |
| `POST` | `/api/v1/moderate_audio_file` | Multipart upload named `file` |
| `GET` | `/api/v1/health` | No body; bearer token still required |

Text example:

```bash
curl -X POST http://localhost:8000/api/v1/moderate_text \
  -H "Authorization: Bearer replace-with-your-user-api-key" \
  -H "Content-Type: application/json" \
  -d '{"text":"Welcome to ACME support. How can I help?"}'
```

Image example:

```bash
curl -X POST http://localhost:8000/api/v1/moderate_image_file \
  -H "Authorization: Bearer replace-with-your-user-api-key" \
  -F "file=@path/to/image.jpg"
```

You can also use the Swagger UI at `http://localhost:8000/docs`: select **Authorize**, enter the `USER_API_KEY` value, choose an endpoint, and select **Try it out**.

## Tests

The test suite checks schemas, all four moderation agents, environment setup, the Gradio interface, and the end-to-end Gemini connection.

Run the tests without the live Gemini connectivity check:

```bash
uv run pytest tests/ -m "not integration" -vv
```

Run the complete suite, including the real Gemini API call:

```bash
uv run pytest tests/ -vv
```

The integration test consumes API quota and requires working credentials and network access.

## Submission evidence

Review artifacts are indexed in [`submission_evidence/README.md`](submission_evidence/README.md). They include automated-verification results, all four Pydantic Eval summaries, a completed Gradio conversation, and readable Phoenix screenshots showing the trace hierarchy, `session.id`, and blocked-turn feedback.

## Evaluations

The eval suites measure moderation quality rather than only code correctness. They combine deterministic checks of the moderation flags with LLM-as-a-judge evaluation of the rationale.

For a quick, lower-cost run, set this in `.env`:

```dotenv
EVAL_NUM_REPEATS=1
```

Run each modality from the repository root:

```bash
uv run evals/text/test_cases.py
uv run evals/image/test_cases.py
uv run evals/audio/test_cases.py
uv run evals/video/test_cases.py
```

The three application services do not need to be running for these scripts. LLM evaluations are non-deterministic, so a score below 100% does not necessarily indicate a code defect. Every eval run calls the model under test and may also call the judge model; increasing `EVAL_NUM_REPEATS` improves the consistency estimate but also increases cost and runtime.

## Observability with Phoenix

After completing a conversation, open [http://localhost:6006/projects](http://localhost:6006/projects) and select the default project. The trace hierarchy includes spans such as:

```text
conversation
└── chat_turn
    ├── moderate_text / moderate_image / moderate_video / moderate_audio
    └── llm_customer
```

Phoenix can help answer questions such as which step failed, where latency occurred, and which moderation flags were returned. Tracing observes the workflow; it does not change the prompt or the model's answer.

## Important limitations

- This project is a learning proof of concept and should not be used as a production safety boundary.
- Moderation decisions are probabilistic and can contain false positives or false negatives.
- API authentication uses one static local bearer token rather than a production identity system.
- The Gradio interface enforces a 5 MB file limit, but the FastAPI upload endpoints do not independently enforce that limit.
- Traces include trainee text, and uploaded media is copied locally to `uploaded_media/` for Phoenix visualization. Do not use real secrets, customer data, or PII; remove retained local media when it is no longer needed.
- Use `uv sync --locked` for the reproducible learning environment. Upgrading major dependencies without a compatibility pass may require code changes.

## Project provenance

ACME Enterprise and the ACME Power Widget Pro are fictional. The exercise sequence and starter scaffolding were supplied as learning materials; this repository contains the completed implementation.
