## Context

`cookidoo-api` is already a Python 3.12 package with an async `Cookidoo` client that covers authentication, localization-aware endpoints, and a large set of recipe/planning workflows. The repository has good existing foundations for a server addition: typed dataclass models, pytest-based tests, and a clear separation between raw HTTP calls and higher-level objects.

What is missing is an MCP-facing integration layer. Today, a caller must import Python code, construct `aiohttp` sessions manually, and know which methods to call. The requested change is a local MCP server over `stdio`, which makes this API usable from MCP clients without changing the underlying Cookidoo library.

Constraints:
- The implementation should reuse the current async client and avoid re-deriving the unofficial API.
- Credentials are sensitive and should stay local to the machine running the `stdio` server.
- The library surface is already large enough that exposing every method as a first-pass MCP tool would create a noisy tool catalog.
- The repository currently has no MCP runtime dependency or server entrypoint.

## Goals / Non-Goals

**Goals:**
- Add a local `stdio` MCP server in Python that wraps the existing `Cookidoo` client.
- Define a predictable configuration model for credentials and localization.
- Provide a curated first tool set for the highest-value Cookidoo workflows already implemented by the library.
- Return structured, model-friendly results and clear recoverable errors instead of leaking raw exceptions.
- Keep the server layer modular so later changes can add more tools or upgrade distribution without rewriting core adapters.

**Non-Goals:**
- Remote HTTP deployment, hosted authentication flows, or multi-user serving.
- UI widgets, elicitation flows, or other rich client features.
- Full one-to-one exposure of every `Cookidoo` method in the first release.
- Premium-only tools and managed-collection tools in the initial release; both are deferred to follow-up changes.
- Reworking the existing `Cookidoo` request logic or changing the upstream API contract.
- Solving secure secret storage beyond standard local process configuration for a `stdio` prototype.

## Decisions

### 1. Use FastMCP in Python for the server runtime

The server will be implemented in Python with `fastmcp`, which fits the existing codebase and supports async tool handlers with minimal adapter code.

Why this choice:
- The repo is already Python-first, so MCP integration can call the current async client directly.
- `fastmcp` keeps the implementation small while still supporting structured tool registration and `stdio` transport.
- Staying in one language avoids a second runtime, an HTTP bridge, or re-implementing Cookidoo behavior in TypeScript.

Alternatives considered:
- Official TypeScript SDK: best spec coverage overall, but it would require either rewriting the Cookidoo client or building a cross-language bridge.
- Hand-rolled JSON-RPC layer: lower dependency count, but significantly more protocol and transport code for no product gain.

### 2. Keep MCP code in a dedicated server package and adapter layer

The server should live beside the existing library, not inside `cookidoo.py`. A dedicated package such as `cookidoo_api/mcp/` should own:
- server bootstrap and tool registration
- configuration loading and validation
- a session/client manager
- response serialization helpers
- exception-to-MCP error translation

Why this choice:
- It preserves the current library as a reusable Python API.
- It keeps MCP concerns isolated from HTTP client logic.
- It makes it easier to test server behavior with mocked client calls.

Alternatives considered:
- Decorating methods directly inside `Cookidoo`: simpler initially, but it couples transport concerns to the domain client and makes both testing and future distribution harder.

### 3. Start with a curated one-tool-per-action surface for core workflows

The first release should stay under the "sweet spot" MCP tool count and focus on the workflows most useful in conversational assistants. The initial tool set should cover:
- localization discovery
- account summary (user info plus subscription state)
- recipe detail lookup
- shopping list snapshot and recipe-based add/remove/clear operations
- custom collection listing/creation/update for recipe planning
- calendar week retrieval and recipe add/remove operations

Why this choice:
- The existing library has 30+ methods; exposing all of them immediately would create a crowded tool catalog and weaken tool selection quality.
- A curated set covers the main planning workflows while keeping descriptions and schemas tight.
- The adapter layer can still be extended later with more dedicated tools or a search/execute pattern if the surface grows.

Alternatives considered:
- Full one-tool-per-method exposure: fastest mapping but too noisy for the initial MCP experience.
- Search + execute from day one: scales better for a very large API, but adds another abstraction layer before there is evidence it is needed for the first usable server.

### 4. Use environment-driven configuration plus a lazy authenticated session manager

The server should read credentials and default localization from environment variables at startup, validate them once, and lazily create an `aiohttp.ClientSession` and `Cookidoo` client when the first authenticated tool is invoked. The session manager should:
- construct `CookidooConfig`
- resolve localization from helper functions
- log in on first use
- refresh tokens before expiry when needed
- close the session on server shutdown

Why this choice:
- `stdio` servers are local processes, so environment-based configuration is the lowest-friction model.
- Lazy initialization keeps read-only startup cheap and avoids failing before a client calls any tool.
- Centralizing auth/session logic avoids repeating login and refresh behavior in every tool handler.

Alternatives considered:
- Passing credentials on every tool call: insecure, repetitive, and hostile to tool selection.
- Prompting interactively for credentials: not reliable across MCP hosts and out of scope for this prototype.

### 5. Return structured content derived from dataclasses and normalize errors

The existing response models are dataclasses, so the server should serialize them with `dataclasses.asdict()` or small DTO helpers and return both human-readable text and structured JSON-compatible content. Aggregated tools such as account summary or shopping list snapshot can compose multiple dataclass payloads into explicit response objects.

Errors from `CookidooConfigException`, `CookidooAuthException`, `CookidooRequestException`, and `CookidooParseException` should be translated into MCP tool errors with recovery hints, for example:
- missing configuration -> explain which env vars are required
- auth failure -> tell the caller to verify credentials/localization
- upstream request failure -> suggest retrying
- parsing failure -> indicate an upstream API shape change

Why this choice:
- Models receive predictable response shapes that are easier to chain across tool calls.
- Error hints keep the transport healthy and help the assistant recover instead of failing generically.

Alternatives considered:
- Returning plain text only: simpler, but harder for downstream tool chaining and follow-up actions.
- Letting exceptions bubble up: risks transport instability and gives poor recovery information.

## Risks / Trade-offs

- [Local credentials in process environment] -> Document the expectation clearly and keep the server local-only for the initial release.
- [Curated tool set omits some existing library features] -> State the initial scope in docs and keep the adapter structure open for follow-up capabilities.
- [Cookidoo upstream API may change without notice] -> Reuse existing exception paths, add focused server tests, and surface parse/request failures as actionable MCP errors.
- [Session reuse bugs could affect multiple tool calls] -> Centralize client lifecycle in one manager and cover login, refresh, and shutdown paths with tests.
- [Collection and calendar workflows span multiple underlying methods] -> Use small adapter functions that hide pagination and object merging from tool handlers.

## Migration Plan

1. Add the MCP runtime dependency and package entrypoint in the Python project metadata.
2. Implement the server package, configuration loader, session manager, serializers, and tool handlers.
3. Add tests for configuration validation, session/auth behavior, and representative tool success/error paths.
4. Document local installation and the required environment variables.
5. Release as an additive change; rollback is simply to stop invoking the MCP entrypoint, since the existing Python library remains unchanged.

## Open Questions

None for the initial implementation. Premium-only tools and managed-collection tools are explicitly out of scope for this change and can be evaluated in later follow-up changes.
