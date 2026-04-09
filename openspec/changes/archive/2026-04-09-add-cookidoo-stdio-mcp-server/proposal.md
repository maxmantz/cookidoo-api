## Why

This repository already reconstructs and wraps a large portion of the Cookidoo mobile API, but it is only consumable as a Python library today. Adding an MCP server makes those capabilities available to MCP-compatible clients immediately, and a local `stdio` transport is the fastest way to turn the existing async client into a usable prototype for personal and team workflows.

## What Changes

- Add a Python MCP server entrypoint that runs over `stdio` and reuses the existing `cookidoo_api.Cookidoo` client instead of reimplementing API calls.
- Introduce a configuration and session model for MCP usage, including credentials, localization selection, login, token refresh, and clean `aiohttp` session lifetime management.
- Expose an initial curated tool set for the highest-value Cookidoo workflows already supported by the library: user/profile info, subscription status, recipe details, shopping-list operations, collection management, and calendar planning.
- Return structured, JSON-friendly tool responses and translate existing library exceptions into clear MCP errors that help the model recover from auth, localization, and request failures.
- Add server-focused tests and user documentation for installation, configuration, and running the MCP server locally.
- Keep the first release intentionally scoped to core planning flows; premium-only and long-tail operations such as custom recipe authoring and every low-level API method can be added in follow-up changes.

## Capabilities

### New Capabilities
- `cookidoo-stdio-server`: Boot and configure a local MCP server over `stdio`, manage Cookidoo authentication, and safely bridge async client calls into MCP tools.
- `cookidoo-core-tools`: Provide a first set of Cookidoo MCP tools for profile, recipe, shopping-list, collection, and calendar workflows.

### Modified Capabilities
- None.

## Impact

- Affected code: new MCP server module/package, CLI entrypoint, configuration helpers, and server-oriented adapters over the existing `Cookidoo` client.
- Dependencies: add a Python MCP framework suitable for local `stdio` transport, most likely `fastmcp`, plus any small supporting config/runtime packages if needed.
- Tests: add focused tests for tool registration, auth/config handling, structured responses, and representative tool flows against mocked client behavior.
- Documentation: update project docs with MCP server usage, required environment variables, and the scope of the initial tool set.
