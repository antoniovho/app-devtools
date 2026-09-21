## About

Describe your API in a way that anyone would quickly understand it, what it is,
what type of API it is, and for what it is used.
Special attention must be paid into describing the **purpose** of the API,
the problem it aims to solve.
Also, make sure to specify the API's **audience**: team, area, or company.

### Main Goal

The bigger business or functional need that justifies the API's existence.
For APIs that have a direct consequence into business applications, this main
goal will ultimately be ensuring the sales or business process. But, for
functional APIs this main goal may be not so straightforward.
Make sure you describe **"why it is important that this API delivers"**.

### Context


If this API is part of a bigger project or ecosystem, explain how this API makes
sense in that bigger context.
Also, briefly describe the other pieces that make up the whole picture.

Provide direct links to these other APIs & tools. It may also be interesting to
specify the API roles: adapter, composite, core, etc.

The use of diagram flows is advised.

### Specifics

Specify to which **protocol** or protocols this API applies.
Also, include any keep-in-mind notes that you believe necessary for describing
the API.

## Usage

Include any notes and specifications (for example, authorizations) the API
consumer needs to bear in mind before invoking this API.
Describe the previous context needed for invoking this API (other APIs that had
to be invoked first) and the following steps that may apply.
The use of flow diagrams is advised.

### Configuration

Describe which **main customizations** this API will allow for and that the API
consumer will like to know at this (initial) stage.
However, keep in mind that the full set of properties or endpoints will be
specified in the Versions tab.

### Example

Provide a simple API's use example (or use case).

## Documentation

You can use this space to include any other information you believe is relevant
and does not fit in any of the previous sections.
More specifically, you may want to:


# API Contracts

This directory contains OpenAPI contracts and reusable API components.

## Shared commons

`commons/` is the shared contract for components that must have the same meaning
across APIs. It is currently maintained in this repository, with its version in
[`commons/rest/VERSION`](commons/rest/VERSION), and included in the bundle of
each consuming API. It does not have an independent release workflow yet.

The shared error contract is available at [../commons/rest/v1/components/errors.yml](../commons/rest/v1/components/errors.yml). It contains both the RFC 7807-compatible error schemas and the reusable OpenAPI HTTP responses.

API-specific technical components remain inside each service. For example, Test Service owns its pagination, sorting fields, UUID, and domain component definitions under `test-service/v1/components`.

Consumers must pin a published API bundle to an immutable API tag or GitHub
Release. The backend must not consume the repository default branch directly.
