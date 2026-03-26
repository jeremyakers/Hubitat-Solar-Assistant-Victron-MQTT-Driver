# AGENTS.md

## Repository scope

- This repository contains a single Hubitat device driver written in Groovy.
- Primary source file: `user_driver_jeremy_akers_Solar_Assistant_Victron_MQTT_Driver_621.groovy`
- There is no `src/`, `test/`, `tests/`, `package.json`, `build.gradle`, or CI workflow in this repo.
- There is no existing Cursor rules file, `.cursorrules`, `.cursor/rules/`, or `.github/copilot-instructions.md` in this repository.

## What this code is

- The driver integrates Hubitat with Solar Assistant / Victron GX devices over MQTT.
- It relies on Hubitat runtime objects such as `interfaces.mqtt`, `device`, `settings`, `state`, `sendEvent(...)`, `schedule(...)`, and `runIn(...)`.

## Files that matter

- `user_driver_jeremy_akers_Solar_Assistant_Victron_MQTT_Driver_621.groovy`: all driver logic, metadata, preferences, MQTT handling, state, and lifecycle hooks.
- `.gitignore`: generic Java/Groovy ignore rules only.
- `LICENSE`: Apache 2.0.

## Build / lint / test commands

This repository currently has no automated build, lint, or test tooling checked in.

### Build

- Build command: none.
- There is no Gradle, Maven, npm, or other build configuration in the repo.
- The deployable unit is the Groovy driver file itself, typically pasted/imported into Hubitat.

### Lint

- Lint command: none.
- There is no Checkstyle, CodeNarc, Spotless, ESLint, Prettier, or equivalent configuration present.

### Test

- Test command: none.
- There are no automated tests, test files, test directories, or test framework configs.

### Running a single test

- Not supported in the current repository state.
- Reason: no test runner or test files exist.
- If a future change adds tests, update this file with the exact command for running one test by file or filter.

## Practical validation workflow

Because there is no build/test harness, validate changes manually and conservatively.

1. Re-read the full affected section of the Groovy file after editing.
2. Confirm Hubitat metadata, commands, attributes, and preferences still line up.
3. Verify MQTT topic names and `settings` references are spelled consistently.
4. Check that each `sendEvent(...)` still emits the intended attribute name and unit.
5. Check that each topic handled in `parse(...)` is also subscribed in `initialize()` when appropriate.
6. Prefer testing on a Hubitat hub with a live MQTT broker when possible.

## High-level structure of the driver

The file follows the common Hubitat driver shape:

1. Header comment with history and provenance
2. Imports
3. `metadata { ... }`
4. `preferences { ... }`
5. Lifecycle and command methods
6. MQTT parse and publish logic
7. Averaging/state helper logic

## Important methods

- `installed()`, `updated()`, `uninstalled()`: lifecycle hooks
- `initialize()`: reset state, connect MQTT, subscribe to topics
- `parse(String description)`: decode inbound MQTT messages and emit events
- `poll()`: keepalive publish and reconnect path
- `sendAverages()`: average accumulated readings and emit events
- `publishMsg(String s)`: outbound MQTT publish helper
- `deviceNotification(String s)`: notification command with JSON-aware handling
- `mqttClientStatus(String status)`, `logsOff()`: status/reconnect and logging helpers

## Code style observed in this repo

These are the conventions already present in the file. Match them unless there is a strong repo-local reason not to.

### Imports

- Keep imports at the top of the file.
- Only two explicit imports are currently used:
  - `groovy.json.JsonSlurper`
  - `groovy.json.JsonOutput`
- Do not add unused imports.

### Formatting

- Use brace-based Groovy style with one top-level method per block.
- Indentation is mixed but predominantly 4 spaces; preserve nearby style when editing small sections.
- Keep blank lines between major method blocks.
- Avoid large reformat-only diffs in this repository.

### Naming

- Method names are mixed: camelCase (`setVersion`, `sendAverages`) and Hubitat-facing snake_case (`set_monitored_battery_soc`).
- Preserve existing public names, command names, attribute names, and preference names exactly.
- Prefer clear camelCase for new internal helpers unless matching an existing Hubitat-facing naming scheme.
- Topic setting names generally use descriptive snake_case such as `power_topic`, `soc_topic`, `min_cell_voltage_topic`.

### Types

- This codebase is dynamically typed and uses `def` heavily.
- Some method parameters are typed, for example `parse(String description)` and `eventProcess(Map evt)`.
- Prefer explicit parameter types only when obvious and consistent with nearby code.

### Hubitat conventions

- Capabilities, commands, and attributes are declared in `metadata.definition`.
- User-configurable inputs are declared in `preferences` via `input ...`.
- Device updates are emitted through `sendEvent(...)`; persistent runtime values live in `state`; user preferences come from `settings`.

### Settings and null-safety

- This file commonly uses safe navigation like `settings?.topicPub` and `settings?.QOS`.
- Continue using safe navigation when reading optional preferences.
- Before substring or numeric conversions, make sure the source setting is expected to exist.

### Error handling

- Do not use empty catches.
- Existing pattern is to log the error and update device status when initialization fails.
- When adding error handling, log enough context to diagnose the MQTT topic/value involved.
- Prefer surfacing failures through logs or `sendEvent(name: "status", value: "error", ...)` when appropriate.

### Logging

- Use Hubitat logging methods: `log.info`, `log.debug`, `log.warn`, `log.error`.
- Gate verbose debug output with `if (logEnable)` when following existing patterns.
- Keep log messages concrete and operationally useful.

### State management

- Arrays used for averaging are reset in `initialize()` and cleared after `sendAverages()`.
- Do not rename or repurpose existing `state` keys casually.
- If you add new `state` keys, initialize them predictably in `initialize()`.

### MQTT behavior

- Publish and subscribe through `interfaces.mqtt` only.
- If you add a new inbound topic branch in `parse(...)`, make sure `initialize()` subscribes to it.
- If you add a new outbound publish path, keep QoS and retained handling consistent with existing helpers.

## Editing guidance for agents

- Make the smallest safe change that solves the task.
- Preserve Hubitat metadata structure and existing command/attribute names.
- Do not rename the main Groovy file unless explicitly requested.
- Do not invent local tooling commands or claim automated tests passed.
- Call out manual validation limits clearly in your summary.

## Good agent workflow here

1. Read the whole Groovy file, not just the target method.
2. Identify all affected MQTT topics, settings, attributes, and state keys.
3. Make a focused edit.
4. Re-read the changed regions plus related lifecycle methods.
5. Report exactly what was changed and what could only be validated manually.

## Known repository constraints

- Single-file repository: cross-cutting edits all happen in one Groovy file.
- No automated quality gates: correctness depends on careful review and manual runtime validation.
- Hubitat runtime APIs are implicit; many symbols will not resolve outside the Hubitat environment.

## When updating this AGENTS.md

Update this file if any of the following are added later:

- automated tests
- a real lint or formatting tool
- a build/deploy script
- Cursor rules
- Copilot instructions
- additional source files or supporting libraries
