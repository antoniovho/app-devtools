# AVH Evalscore API

AVH Evalscore is an experimental OpenAPI REST contract for authoring
reproducible LLM evaluations, scheduling their asynchronous execution, and
inspecting immutable results and invocation traces.

This directory contains contract source only. Backend implementation, generated
server stubs, generated clients, Python tooling, and virtual environments belong
to consumer repositories.

## Contract Source

- Root document: `openapi-rest.yml`
- API metadata: `metadata.yml`
- Current version: `0.1.0`
- Base path: `/v1`
- Local server: `http://localhost:8080`
- Authentication: Bearer JWT (`bearerAuth`)

The root document references service and component files under `v1/`. Bundle the
contract before distributing it or passing it to code generators.

## API Scope

| Area            | Resources                                                     |
| --------------- | ------------------------------------------------------------- |
| Evaluations     | Evaluation definitions and single-turn, multi-turn, and trajectory cases. |
| Evaluation Runs | Asynchronous runs, immutable result attempts, and cancellation requests. |

All operations are secured by the global Bearer JWT requirement. Error responses
use `application/problem+json` and the shared RFC 9457 Problem Details schema
from `apis/commons/rest/v1/components/errors.yml`.

## Execution Rules

`POST /v1/evaluations/{evaluationId}/runs` captures the current evaluation
definition and its cases, then returns `202 Accepted` while execution proceeds
asynchronously. The resulting run retains its immutable snapshot, and each
result attempt records the target output, score, reasoning, and optional trace.

`POST /v1/evaluation-runs/{runId}/cancellations` requests cancellation only for
runs that have not reached a terminal state.

## Validate Locally

Run from the repository root:

```bash
npx --yes @redocly/cli@1.34.20 lint apis/avh-evalscore/rest/openapi-rest.yml

npx --yes @redocly/cli@1.34.20 bundle \
  apis/avh-evalscore/rest/openapi-rest.yml \
  --output /tmp/avh-evalscore-api-0.1.0.yml

openapi-generator-cli validate -i /tmp/avh-evalscore-api-0.1.0.yml
```

The bundle is a single OpenAPI YAML document with resolved references. It is not
generated server code or a client library.
