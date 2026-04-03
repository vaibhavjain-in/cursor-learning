# PyCharm → Cursor migration

Cursor is built on the same editor core as VS Code, so most extensions and `settings.json` / `launch.json` patterns apply directly. The shift is from JetBrains’ **integrated** defaults to a **modular** setup—plus Cursor’s AI features on top.

---

## 1. JetBrains feel (shortcuts and UI)

- **Keybindings:** Install **[IntelliJ IDEA Keybindings](https://marketplace.visualstudio.com/items?itemName=k--kato.intellij-idea-keybindings)** (publisher **k--kato**). It maps familiar combos such as double-`Shift` search, `⌥ Enter` quick fixes, and much of PyCharm’s navigation. Optional: the extension can help import a custom IntelliJ keymap on macOS.
- **Theme:** Use **[Darcula Theme](https://marketplace.visualstudio.com/items?itemName=monokai.theme-darcula)** or another “Darcula / IntelliJ” theme so colors stay familiar.
- **Activity bar:** PyCharm users often dislike the vertical icon strip. Right‑click it and use **Move Activity Bar to Top** (or hide it) to reclaim space.

---

## 2. Extension stack (Python + parity with PyCharm)

| Extension | Role |
| :--- | :--- |
| **Python** (Microsoft) | Interpreter, debugging, testing integration—core Python support. |
| **Pylance** | Fast IntelliSense, types, go-to-definition / references (closest to PyCharm’s analysis). |
| **Ruff** | Lint + format in one tool; replaces separate Black / Flake8 / isort style setups for many projects. |
| **GitLens** | File history, blame, richer Git context (closest to PyCharm’s VCS tooling). |
| **Error Lens** | Inline error/warning text on the line (strong gutter feedback). |
| **Test Explorer UI** + **Python Test Explorer** | Sidebar test tree and runs for `pytest` / `unittest`. |

Install what you need; the first four are the usual “non‑negotiable” set for a PyCharm‑like Python workflow.

---

## 3. Environment and `.env`

PyCharm wires run configs and envs through the IDE UI. In Cursor you declare paths in settings and launch configs.

### Workspace or user `settings.json`

**Command Palette** → `Preferences: Open User Settings (JSON)` (global) or edit `.vscode/settings.json` (project).

```json
{
  "python.envFile": "${workspaceFolder}/.env",
  "python.terminal.activateEnvInCurrentTerminal": true,
  "python.terminal.activateEnvironment": true,
  "terminal.integrated.env.osx": {
    "PYTHONPATH": "${workspaceFolder}"
  }
}
```

On **Linux**, replace `terminal.integrated.env.osx` with `terminal.integrated.env.linux`. On **Windows**, use `terminal.integrated.env.windows`. Adjust `PYTHONPATH` only if your project needs it.

### Debugger `launch.json`

In `.vscode/launch.json`, point the debugger at your `.env`:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Python: Current File",
      "type": "python",
      "request": "launch",
      "program": "${file}",
      "console": "integratedTerminal",
      "envFile": "${workspaceFolder}/.env"
    }
  ]
}
```

Use **Run and Debug** → create a launch configuration from the template if yours differs (extension updates occasionally change suggested fields).

### Interpreter

**Status bar** (bottom right) → click the Python version → pick your venv or conda env. Or **Command Palette** → `Python: Select Interpreter`.

---

## 4. Ruff as default formatter (optional)

Align with PyCharm’s “reformat on save” by setting Ruff as formatter and enabling format on save (names may vary slightly by Ruff extension version):

```json
{
  "[python]": {
    "editor.defaultFormatter": "charliermarsh.ruff",
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
      "source.fixAll.ruff": "explicit",
      "source.organizeImports.ruff": "explicit"
    }
  }
}
```

---

## 5. Migration checklist

1. Install **Cursor**; if offered, import settings/extensions from VS Code.
2. Install **IntelliJ IDEA Keybindings** and a **Darcula** theme.
3. **Open the project folder** (`File` → `Open Folder`)—workspace‑centric, not a `.ipr` file.
4. **Select interpreter** so Pylance can index (`Python: Select Interpreter`).
5. Install **Python**, **Pylance**, **Ruff**, **GitLens**, **Error Lens**; add test explorer extensions if you rely on the test tree.
6. Add `.vscode/settings.json` (env + optional Ruff) and **`launch.json`** for debugging.
7. Turn on **Auto Save** if you prefer PyCharm’s always‑saved feel (`File` → `Auto Save`).
8. Open the **integrated terminal** and confirm the venv activates and important env vars resolve (`echo …`).
9. **Cursor:** index the codebase or add folders to context so AI features see your project (per current Cursor UI).
10. Run **F5** (debug) on a small script to confirm breakpoints and `.env` loading.

---

## 6. PyCharm → Cursor quick translation

Shortcuts below assume the **IntelliJ keymap** where noted; Cursor/VS Code defaults differ—check **Keyboard Shortcuts** in settings.

| PyCharm | In Cursor / VS Code |
| :--- | :--- |
| Search Everywhere | Double `Shift` (IntelliJ map) or `⌘ P` / `⌘ Shift P` |
| Project interpreter | Status bar Python version or `Python: Select Interpreter` |
| Run / Debug configurations | `.vscode/launch.json`; **F5** to start debugging |
| Find in path | `⌘ Shift F` (global search) |
| Quick fix / intention | `⌥ Enter` (IntelliJ map) |
| Structure / outline | Explorer outline or symbol search (`⌘ Shift O` with IntelliJ map, varies) |
| Terminal | `` Ctrl ` `` (default) or your keymap |
| Rename | `Shift F6` (IntelliJ) or `F2` (VS Code default) |

### Cursor‑specific (AI)

| Goal | Typical entry points |
| :--- | :--- |
| Chat | `⌘ L` or Chat in the sidebar (verify in Keyboard Shortcuts) |
| Inline edit | `⌘ K` (common default; confirm in app) |
| Composer (multi‑file edits) | `⌘ I` or Command Palette → Composer—shortcuts evolve; use **Cursor Settings → Keyboard Shortcuts** as source of truth |

Composer is useful for changes that span several files—closer to a structured task than single‑file refactor tools in classic PyCharm.

---

## 7. Sanity checks

- After keymap install, try **double Shift** and an **`⌥ Enter`** quick fix on a warning.
- Confirm **Ruff** and **Pylance** are active for the same interpreter you run in the terminal.
- If `.env` values are missing in the terminal, re‑check `python.envFile` and shell‑specific `terminal.integrated.env.*` blocks.

This doc merges overlapping VS Code–oriented and Cursor‑oriented migration notes into one flow; adjust extension IDs and generated `launch.json` if your Python extension version differs.
