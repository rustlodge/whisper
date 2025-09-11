# Whisper Project – Rust Code Styleguide

This document defines the **coding standards and style conventions** for the Whisper project. Gemini code assist must follow these rules when generating or refactoring code.

---

## 1. Project Organization

* **Modules**

  * Project is split into two root modules:

    * `whisper_core`: domain logic, models, services, error handling.
    * `whisper_ui`: CLI, command handling, progress indicators.
  * Each feature belongs in its respective submodule (`*_service`, `*_command`, `indicators`).
* **File naming**

  * Use **snake\_case** for file/module names (e.g., `whisper_notifyme_service.rs`).
* **Struct and Enum placement**

  * Public configuration/data models go under `whisper_core::models`.
  * Errors are defined in `whisper_core::exceptions`.

---

## 2. Naming Conventions

* **Modules & files**: `snake_case`.
* **Structs, Enums, Traits**: `PascalCase` (e.g., `WhisperConfig`, `WhisperError`).
* **Functions**: `snake_case` (e.g., `load_configuration_file`, `validate_file`).
* **Error enum variants**: `PascalCase` (e.g., `ConfigFileNotFound`, `NetworkRequestError`).
* **Command enums**: `PascalCase` (e.g., `WhisperCommandMenu::NotifyMe`).
* **Configuration fields**: `snake_case` in Rust, mapped via `serde` aliases when needed.

---

## 3. Error Handling

* Use [`thiserror`](https://docs.rs/thiserror) for `WhisperError`.
* Wrap third-party errors transparently using `#[from]`.
* Define descriptive, domain-specific variants:

  ```rust
  #[error("File {file_path:?} has incorrect extension: Expected {expected_extension:?}")]
  IncorrectFileExtension { file_path: String, expected_extension: String }
  ```
* Avoid panics in production code. Use `Result<T, WhisperError>`.

---

## 4. CLI Design (Clap)

* Use [`clap`](https://docs.rs/clap) `derive` API for CLI definitions.
* Define a root struct `WhisperCommandApp` with:

  * Global flags (e.g., `--config`).
  * A `commands` field of type `WhisperCommandMenu`.
* Subcommands follow this format:

  * `Init`: configuration file creation.
  * `NotifyMe`: notifications to Echo devices.
  * `TestFairy`: binary upload to TestFairy.
* Always provide:

  * `about` (short description).
  * `after_help` (usage hints).
  * Explicit argument attributes (`index`, `short`, `long`).

---

## 5. Configuration Management

* Store configurations in `whisper.toml`.
* Use [`serde`](https://serde.rs/) for `Deserialize`/`Serialize`.
* Provide default values via `#[derive(Default)]` and explicit `new()` constructors.
* File handling rules:

  * If config file is missing, return `WhisperError::ConfigFileNotFound`.
  * If a section is missing, return `WhisperError::ConfigMissingDefinition`.

---

## 6. Services

* Each external integration (NotifyMe, TestFairy, etc.) has its own `*_service.rs`.
* Services:

  * Validate inputs (`PathBuf`, tokens, extensions).
  * Handle retries where applicable (e.g., TestFairy upload).
  * Log progress to stdout/stderr (no silent failures).
* Common patterns:

  * `fn handle(...) -> Result<(), WhisperError>` for main entrypoints.
  * Keep external library usage inside services, not CLI.

---

## 7. Progress Indicators

* Use [`indicatif`](https://docs.rs/indicatif) for progress bars.
* Create wrapper structs (`ProgressReader`) to integrate with I/O.
* Always provide human-readable formatting (`{bar}`, `{eta}`, `{bytes}`).

---

## 8. Testing

* Unit tests live in `#[cfg(test)] mod tests { ... }` inside the module they test.
* Use [`tempfile`](https://docs.rs/tempfile) for file-system tests.
* Use [`mockito`](https://docs.rs/mockito) for HTTP mocks.
* Validate:

  * Success paths (valid files, successful uploads).
  * Failure cases (timeouts, missing configs, invalid extensions).
* Prefer `assert!(matches!(...))` for error variant checks.

---

## 9. Dependencies

* **clap** → CLI argument parsing.
* **serde / toml** → configuration serialization.
* **reqwest (blocking)** → HTTP requests.
* **indicatif** → progress bars.
* **color-eyre** → error reporting.
* **thiserror** → error definitions.
* **mockito / tempfile** (dev) → testing support.

---

## 10. General Guidelines

* Always prefer **explicit imports** over glob imports (`use crate::module::submodule::*` is avoided except for prelude).
* Keep CLI logic (`whisper_ui`) thin; delegate business logic to services in `whisper_core`.
* Follow **functional, immutable-first style** where possible.
* Document public APIs with `///` doc comments.
