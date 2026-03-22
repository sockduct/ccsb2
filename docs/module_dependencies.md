# Module Dependency Map

```mermaid
flowchart LR
    subgraph entry["Entry Points"]
        MAIN["__main__.py"]
        SERVE["scripts/serve_docs.py"]
    end

    subgraph core["Core Package (second_brain)"]
        INIT["__init__.py"]
        CLI["cli.py"]
        APP["app.py"]
        NOTES["notes.py"]
    end

    subgraph tests["Tests"]
        CONFTEST["conftest.py"]
        T_APP["test_app.py"]
        T_CLI["test_cli.py"]
        T_NOTES["test_notes.py"]
    end

    subgraph ext["External Dependencies"]
        CLICK["click"]
        LOGURU["loguru"]
        PYTEST["pytest"]
        STDLIB["stdlib\n(pathlib, re, os,\ndatetime, sys, subprocess)"]
    end

    %% Entry point imports
    MAIN -->|"imports cli"| CLI

    %% Core internal imports
    CLI -->|"imports configure_logging"| APP
    CLI -->|"imports create_note"| NOTES

    %% Core external imports
    CLI --> CLICK
    APP --> LOGURU
    APP --> STDLIB
    CLI --> STDLIB
    NOTES --> STDLIB
    SERVE --> STDLIB

    %% Test imports (local)
    T_APP -->|"imports console_format, main"| APP
    T_CLI -->|"imports cli"| CLI
    T_NOTES -->|"imports slugify, build_note_path, create_note"| NOTES

    %% Test external imports
    T_APP --> PYTEST
    T_CLI --> CLICK
    T_NOTES --> PYTEST
    CONFTEST --> PYTEST
```

## Dependency Summary

| Module | Imports From | Safe to Change? |
|--------|-------------|-----------------|
| `notes.py` | stdlib only | Yes — no internal dependents except `cli.py` and `test_notes.py` |
| `app.py` | loguru, stdlib | Yes — only `cli.py` and `test_app.py` depend on it |
| `cli.py` | `app`, `notes`, click | Changing its public API breaks `__main__.py` and `test_cli.py` |
| `__main__.py` | `cli` | Yes — nothing imports from it |
| `scripts/serve_docs.py` | stdlib only | Yes — fully isolated |
| `conftest.py` | pytest | Yes — fixtures affect all tests but no module imports it |

## No Circular Dependencies

The dependency graph is a clean DAG (directed acyclic graph). All data flows in one direction:

```
tests → cli → app
              ↘
        notes
```
