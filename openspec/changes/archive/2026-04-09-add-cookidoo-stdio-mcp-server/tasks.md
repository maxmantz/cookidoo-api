## 1. Project Setup

- [x] 1.1 Add the MCP runtime dependency and package entrypoint configuration in `pyproject.toml`
- [x] 1.2 Create the `cookidoo_api/mcp/` package structure for server bootstrap, config loading, session management, serialization, and tool handlers

## 2. Server Foundation

- [x] 2.1 Implement environment-based MCP configuration loading that validates credentials and resolves the default Cookidoo localization
- [x] 2.2 Implement a shared async Cookidoo session manager that performs lazy login, token refresh, and clean `aiohttp` shutdown
- [x] 2.3 Implement serializer and error-mapping helpers that convert dataclass models into structured MCP responses and known library exceptions into recoverable tool errors
- [x] 2.4 Bootstrap the FastMCP `stdio` server and register shared tool metadata, descriptions, and annotations

## 3. Read Tooling

- [x] 3.1 Implement `cookidoo_list_localizations`, `cookidoo_get_account_summary`, and `cookidoo_get_recipe_details`
- [x] 3.2 Implement `cookidoo_get_shopping_list` with structured recipe-linked and additional-item data
- [x] 3.3 Implement `cookidoo_list_custom_collections` and `cookidoo_get_calendar_week`

## 4. Mutation Tooling

- [x] 4.1 Implement `cookidoo_add_recipe_ingredients_to_shopping_list`, `cookidoo_remove_recipe_ingredients_from_shopping_list`, and `cookidoo_clear_shopping_list`
- [x] 4.2 Implement `cookidoo_create_custom_collection`, `cookidoo_add_recipes_to_custom_collection`, and `cookidoo_remove_recipe_from_custom_collection`
- [x] 4.3 Implement `cookidoo_add_recipes_to_calendar` and `cookidoo_remove_recipe_from_calendar`

## 5. Verification and Documentation

- [x] 5.1 Add pytest coverage for configuration validation, authenticated session lifecycle, structured serialization, and MCP error mapping
- [x] 5.2 Add pytest coverage for representative read and mutation tools using mocked `Cookidoo` client behavior
- [x] 5.3 Document local installation, required environment variables, available MCP tools, and `stdio` startup in project documentation
