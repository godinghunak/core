# Config Flow Platform Knowledge

## Overview
Config flows handle the user-facing setup and configuration of integrations in Home Assistant. They provide a structured way to collect credentials, options, and settings through the UI.

## Key Components

### `config_entries.py`
- `ConfigFlow`: Base class for integration setup flows
- `OptionsFlow`: Base class for modifying existing integration options
- `ConfigFlowResult`: TypedDict representing flow step results

## Common Patterns

### Basic Config Flow
```python
class MyIntegrationConfigFlow(ConfigFlow, domain=DOMAIN):
    VERSION = 1

    async def async_step_user(
        self, user_input: dict[str, Any] | None = None
    ) -> ConfigFlowResult:
        errors: dict[str, str] = {}
        if user_input is not None:
            try:
                await validate_input(self.hass, user_input)
            except CannotConnect:
                errors["base"] = "cannot_connect"
            except InvalidAuth:
                errors["base"] = "invalid_auth"
            else:
                return self.async_create_entry(
                    title=user_input[CONF_HOST], data=user_input
                )
        return self.async_show_form(
            step_id="user",
            data_schema=STEP_USER_DATA_SCHEMA,
            errors=errors,
        )
```

### Unique ID Handling
Always set a unique ID to prevent duplicate entries:
```python
await self.async_set_unique_id(device_unique_id)
self._abort_if_unique_id_configured()
```

### Re-authentication Flow
```python
async def async_step_reauth(
    self, entry_data: Mapping[str, Any]
) -> ConfigFlowResult:
    return await self.async_step_reauth_confirm()

async def async_step_reauth_confirm(
    self, user_input: dict[str, Any] | None = None
) -> ConfigFlowResult:
    ...
    return self.async_update_reload_and_abort(
        self._get_reauth_entry(), data_updates=user_input
    )
```

## Quality Scale Requirements
- **bronze**: Config flow must exist for UI-configurable integrations
- **silver**: Must handle re-authentication (`async_step_reauth`)
- **gold**: Must support options flow if runtime settings exist
- **platinum**: Full test coverage including error paths

## Testing
Use `hass.config_entries.flow.async_init` and `hass.config_entries.flow.async_configure` in tests. Mock external calls with `patch`.
