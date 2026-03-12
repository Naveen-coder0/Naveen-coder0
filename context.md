# NDJL Project — Complete Context

## What is NDJL?

**NDJL** stands for **Natural Developer Junior Language** (also referenced as *Natural Developer Journey Language* in older notes). It is a custom, beginner-friendly programming language designed so that code reads as close to plain English as possible. The motto is **"Write Code the Way Humans Think."**

NDJL is interpreted (not compiled to machine code) and runs on top of Node.js. The entire project is split into three main components:

1. **The NDJL Interpreter** — The language runtime (`NDJL/`)
2. **The VS Code Extension** — Editor tooling (`ndjl-extension/`)
3. **The Windows Installer** — Distribution packaging (`installer/`)
4. **The Marketing Website** — A Next.js landing page (`websitye/`)

---

## 1. The NDJL Language (`NDJL/`)

### How It Works

The interpreter follows a classic three-stage pipeline:

```
Source (.ndjl file)
    ↓ tokenize()    [tokenizer.js]
Token stream
    ↓ Parser.parse() [parser.js + ast.js]
Abstract Syntax Tree (AST)
    ↓ Interpreter.interpret() [interpreter.js]
Output (stdout / stdin)
```

### Entry Points

- `NDJL/index.js` — Direct Node.js entry: `node index.js run <file.ndjl>` or `node index.js <file.ndjl>`
- `NDJL/bin/ndjl.js` — CLI shebang entry: `ndjl run <file.ndjl>` or `ndjl <file.ndjl>`

Both accept the same syntax; the `bin` version is what the npm global installation or compiled `.exe` uses.

### Tokenizer (`NDJL/tokenizer.js`)

Converts raw source text into a flat array of typed tokens.

**Token types:**

| Token Type  | Examples                        |
|-------------|--------------------------------|
| `KEYWORD`   | `create`, `say`, `if`, `else`, `end`, `define`, `return`, `ask`, `into`, `repeat`, `times`, `and`, `or`, `not`, `greater`, `less`, `equals`, `int`, `str`, `number`, `text`, `list` |
| `IDENTIFIER`| Any user-defined variable/function name |
| `NUMBER`    | `42`, `3.14`                   |
| `STRING`    | `"hello world"`                |
| `OPERATOR`  | `+`, `-`, `*`, `/`, `%`       |
| `EQUALS`    | `=`                            |
| `LPAREN`    | `(`                            |
| `RPAREN`    | `)`                            |
| `COMMA`     | `,`                            |
| `LBRACKET`  | `[`                            |
| `RBRACKET`  | `]`                            |
| `DOT`       | `.`                            |
| `EOF`       | end of file sentinel           |

Comments use `#` and are stripped during tokenization (the tokenizer simply skips them — they never produce a token because the character scanner skips unknown characters after operators; actually `#` would cause an "Unknown character" error unless the reader notes that tokenizer skips `#` lines — upon closer reading, the tokenizer would error on `#`. The `test.ndjl` file uses `#` comments, which works because those lines are commented out with a leading `#` and the tokenizer skips whitespace/newlines but NOT `#`. This means `#` comments in source code would actually throw an error at the tokenizer unless the parser or calling code cleans them first — or it is an intended limitation noted by the test file using commented-out blocks).

> **Note:** Reviewing the tokenizer carefully — `#` is NOT handled as a comment by the tokenizer. The `test.ndjl` file uses `#` as comments but those blocks would throw `Unknown character` at runtime. This appears to be a known limitation / work-in-progress in the language.

### Parser (`NDJL/parser.js`) and AST (`NDJL/ast.js`)

Converts the token stream into a tree of AST nodes. Each node type maps to an NDJL language construct.

**AST Node Types:**

| Node Type           | Description                                          |
|---------------------|-----------------------------------------------------|
| `Program`           | Root node; holds array of top-level statements      |
| `VarDeclaration`    | `create <type> <name> = <expr>`                     |
| `Assignment`        | `<name> = <expr>` (reassignment of existing var)   |
| `IndexAssignment`   | `<name>[<index>] = <expr>`                         |
| `SayStatement`      | `say <expr>` — prints to stdout                    |
| `AskStatement`      | `ask "<prompt>" into <var>` — reads stdin           |
| `IfStatement`       | `if <condition> ... else ... end`                  |
| `RepeatStatement`   | `repeat <n> times ... end`                         |
| `FunctionDeclaration` | `define <name>(<params>) ... end`               |
| `ReturnStatement`   | `return <expr>`                                    |
| `CallExpression`    | `<name>(<args>)` — function call                   |
| `BinaryExpression`  | arithmetic (`+`, `-`, `*`, `/`, `%`) and comparison (`>`, `<`, `==`) |
| `LogicalExpression` | `and`, `or`                                        |
| `UnaryExpression`   | `not`                                              |
| `Identifier`        | Variable reference                                 |
| `NumberLiteral`     | Numeric constant                                   |
| `StringLiteral`     | String constant                                    |
| `ArrayLiteral`      | `[elem1, elem2, ...]`                              |
| `MemberExpression`  | `<array>[<index>]`                                 |
| `PropertyAccess`    | `<var>.length`                                     |

### Interpreter (`NDJL/interpreter.js`)

Tree-walk interpreter. Maintains:
- `globals` — global variable scope (plain object)
- `env` — current scope (globals, or a local function scope that inherits from globals via `Object.create`)
- `functions` — registered function declarations

**Built-in behaviors:**
- `say` → `console.log(value)`
- `ask "..." into varName` → synchronous stdin read via `fs.readSync(0, ...)`, stores trimmed string
- Type enforcement on declaration: `int`/`number` must be numeric; `str`/`text` must be string; `list` must be array
- Default values on uninitialized declaration: `int`/`number` → `0`, `str`/`text` → `""`, `list` → `[]`
- Function calls create a fresh local scope; `return` is implemented by throwing a `ReturnValue` exception caught in the call handler
- Division by zero throws a runtime error
- `not` logical operator, `and`/`or` short-circuit logical operators

### NDJL Language Syntax Summary

```ndjl
# Variable declaration
create number age = 25
create text name = "Alice"
create list scores = [90, 85, 78]

# Output
say "Hello, World!"
say age

# Input
ask "Enter your name: " into name

# Arithmetic
create number result = age + 5
create number product = age * 2

# Comparison & conditionals
if age greater than 18
  say "Adult"
else
  say "Minor"
end

if age equals 25
  say "Exactly 25"
end

if age less than 30
  say "Under 30"
end

# Logical operators
if age greater than 18 and age less than 65
  say "Working age"
end

if not age equals 0
  say "Non-zero age"
end

# Loops
repeat 5 times
  say "Hello"
end

# Functions
define greet(name)
  say "Hello " + name
end

greet("Alice")

define add(a, b)
  return a + b
end

create number sum = add(3, 4)

# List access
create number first = scores[0]
scores[1] = 99

# Property access
create number len = scores.length

# Reassignment
age = 26
```

**Variable types:**
- `int` / `number` — numeric (integer or float)
- `str` / `text` — string
- `list` — array

---

## 2. The VS Code Extension (`ndjl-extension/`)

A full VS Code language extension for `.ndjl` files published under the `ndjl` publisher ID.

### Features

| Feature | Details |
|---------|---------|
| **Syntax Highlighting** | TextMate grammar in `syntaxes/ndjl.tmLanguage.json`. Keywords (control: purple), other keywords (blue), operators (blue), strings (green/yellow), numbers (orange), comments (`#`, gray), array brackets |
| **Autocomplete** | Registers a `CompletionItemProvider` for all NDJL keywords with descriptions. Also provides short-trigger shortcuts: `cr` → `create`, `sa` → `say`, `re` → `repeat`, `de` → `define`, `nu` → `number`, `te` → `text`, `li` → `list`, `as` → `ask` |
| **Code Snippets** | File: `snippets/ndjl.json`. Prefixes: `if`, `ifelse`, `repeat`, `define`, `crnum`, `crtxt`, `crlist`, `say`, `ask` |
| **Language Configuration** | `language-configuration.json`: line comment `#`, auto-closing pairs for `[]`, `()`, `""`, `''`, code folding on `if`/`repeat`/`define` ... `end`, auto-indent rules |
| **Run Command** | Command `ndjl.runFile` — saves the active file, opens a new integrated terminal, and runs the file. Appears in the editor right-click context menu and in the title bar run button when a `.ndjl` file is open |
| **Interpreter Path Config** | Setting `ndjl.interpreterPath` — absolute path to `index.js`. If not set, auto-detects `NDJL/index.js` in the workspace or falls back to the global `ndjl` CLI |

### Extension Activation

Activates on:
- `onLanguage:ndjl` (any `.ndjl` file is opened)
- `onCommand:ndjl.runFile`

### Run Command Resolution Order

1. Custom `ndjl.interpreterPath` setting → `node "<path>" "<file>"`
2. Auto-detect `NDJL/index.js` in any workspace folder → `node "<ndjl/index.js>" "<file>"`
3. Fallback → `ndjl run "<file>"` (global CLI)

---

## 3. The Windows Installer (`installer/`)

A full production installer pipeline for distributing NDJL on Windows.

### Build Prerequisites

| Tool | Purpose |
|------|---------|
| Node.js 18+ | Runtime |
| `@yao-pkg/pkg` (global) | Bundle `NDJL/index.js` into a standalone `ndjl.exe` (no Node.js required on target machine) |
| `@vscode/vsce` (global) | Package the VS Code extension into a `.vsix` file |
| Inno Setup 6 | Compile the final Windows installer (`NDJLSetup.exe`) |

### Build Process (`installer/build.ps1`)

Runs in 5 stages:

1. **Generate assets** — runs `generate-wizard-images.js` (creates `wizard-large.bmp` 164×314 and `wizard-small.bmp` 55×55 for the Inno Setup wizard UI) and `generate-icon.js` (creates `ndjl.ico` / `ndjl-logo.ico`, a 32×32 icon with a stylized "N" letter using a purple-to-cyan gradient)
2. **Package runtime** — `pkg index.js --target node18-win-x64 --output dist/ndjl.exe --compress GZip`
3. **Package extension** — `vsce package --out dist/ndjl-extension.vsix`
4. **Copy sample** — copies `NDJL/test.ndjl` → `dist/example.ndjl`
5. **Compile installer** — runs Inno Setup 6 (`ISCC.exe`) on `ndjl-setup.iss`

### Installer (`installer/ndjl-setup.iss`)

An Inno Setup 6 script that builds `NDJLSetup.exe`. What it installs:

| Action | Details |
|--------|---------|
| Copies `ndjl.exe` | To `C:\Program Files\NDJL\` |
| Copies extension `.vsix` | To `C:\Program Files\NDJL\` |
| Copies `example.ndjl` | To `C:\Program Files\NDJL\examples\` |
| Adds to system PATH | Optional task (checked by default) |
| `.ndjl` file association | Opens with `ndjl.exe run "%1"` |
| Context menu entry | "Run NDJL File" right-click on `.ndjl` files |
| Installs VS Code extension | Runs `code --install-extension ndjl-extension.vsix` |
| Start Menu shortcuts | NDJL Command Prompt, Examples folder, Uninstall |
| Desktop shortcut | Optional (unchecked by default) |
| Full uninstaller | Registered via Windows Add/Remove Programs |

**Installer UI:** Premium dark theme with electric blue (`#2078FF`) accent, deep void background (`#0F0F28`), custom wizard panel images, requires Windows 10+, admin privileges, 64-bit.

### Asset Generators

- **`generate-icon.js`** — Pure Node.js ICO file writer (no external deps). Produces a 32×32 32-bit RGBA `.ico` with a letter "N" using a gradient pixel pattern (purple top → cyan bottom) on a dark `#0F0F28` background.
- **`generate-wizard-images.js`** — Pure Node.js 24-bit BMP writer (no external deps). Produces two BMP images for the Inno Setup wizard:
  - `wizard-large.bmp` (164×314) — Left sidebar on Welcome/Finish pages: deep purple-to-blue-to-teal diagonal gradient, glowing orbs, "NDJL" wordmark and "1.0" version in a custom 5×7 pixel font
  - `wizard-small.bmp` (55×55) — Top-right on inner pages: condensed version of the same design

---

## 4. The Marketing Website (`websitye/project-bolt-sb1-ljmv3jfg/project/`)

A **Next.js 13+ (App Router)** landing page website for NDJL built with TypeScript, Tailwind CSS, shadcn/ui components, and Framer Motion animations. Configured for deployment on **Netlify**.

### Tech Stack

| Technology | Role |
|------------|------|
| Next.js (App Router) | Framework |
| TypeScript | Language |
| Tailwind CSS | Styling |
| shadcn/ui | Component library (full set installed) |
| Framer Motion | Scroll-triggered entrance animations |
| Lucide React | Icons |
| Netlify | Hosting (`netlify.toml` present) |

### Page Structure (`app/page.tsx`)

A single-page app. Sections rendered in order:

| Section | Component | Description |
|---------|-----------|-------------|
| Hero | `Hero.tsx` | Full-screen hero with animated particle background, "NDJL" large gradient heading, "Natural Developer Junior Language" subtitle, "Write Code the Way Humans Think" tagline, Get Started / Documentation / GitHub buttons, animated scroll indicator |
| About | `About.tsx` | What NDJL is and its mission |
| Why NDJL | `WhyNDJL.tsx` | Reasons to choose NDJL over other languages |
| Code Example | `CodeExample.tsx` | Live code showcase with syntax-highlighted NDJL snippet, copy-to-clipboard button |
| Features | `Features.tsx` | 5-card grid: Simple Syntax, AI Friendly, Fast Learning Curve, Clean Structure, Developer Productivity |
| Installation | `Installation.tsx` | 3-step terminal commands: `npm install ndjl`, `ndjl run app.ndjl`, `ndjl create my-project` |
| Documentation | `Documentation.tsx` | 5-card doc index: Variables, Conditions, Lists, Functions, Modules |
| Team | `Team.tsx` | Team/contributors section |
| Community | `Community.tsx` | Community links / call to action |
| Footer | `Footer.tsx` | Footer with links |

### Global Styling (`app/globals.css`)

Dark space theme (`bg-[#0f172a]` — deep navy) with:
- Glassmorphism cards (`.glassmorphism` utility: `bg-white/5 backdrop-blur border border-white/10`)
- Glow effects (`.glow-text`, `.glow-cyan`)
- Animated particle background (`ParticleBackground.tsx` component)

---

## 5. Project-Wide Information

### License

MIT License — Copyright (c) 2026 NDJL Project

### Version

`1.0.0` across all components (interpreter, extension, installer)

### Repository Layout

```
NDJL-EXTENSON-COMPILER/
├── NDJL/                          # Language interpreter (Node.js)
│   ├── tokenizer.js               # Lexer — source → tokens
│   ├── parser.js                  # Parser — tokens → AST
│   ├── ast.js                     # AST node class definitions
│   ├── interpreter.js             # Tree-walk interpreter
│   ├── index.js                   # CLI entry point
│   ├── test.ndjl                  # Sample/test NDJL program
│   ├── package.json               # npm package + pkg config
│   └── bin/
│       └── ndjl.js                # Shebang CLI wrapper
│
├── ndjl-extension/                # VS Code extension
│   ├── extension.js               # Extension activation, commands, completions
│   ├── package.json               # Extension manifest
│   ├── language-configuration.json# Brackets, comments, folding, indentation
│   ├── README.md                  # Extension documentation
│   ├── snippets/
│   │   └── ndjl.json              # Code snippets
│   └── syntaxes/
│       └── ndjl.tmLanguage.json   # Syntax highlighting grammar
│
├── installer/                     # Windows distribution pipeline
│   ├── build.ps1                  # PowerShell build automation script
│   ├── build.bat                  # Batch wrapper (alternative)
│   ├── BUILD.md                   # Build instructions
│   ├── ndjl-setup.iss             # Inno Setup installer script
│   ├── generate-icon.js           # Generates ndjl.ico programmatically
│   ├── generate-wizard-images.js  # Generates installer wizard BMP images
│   └── assets/
│       └── LICENSE.txt            # MIT license text
│
├── websitye/                      # Marketing website (Next.js)
│   └── project-bolt-sb1-ljmv3jfg/
│       └── project/               # Full Next.js project
│           ├── app/               # App Router pages
│           ├── components/
│           │   ├── sections/      # Page sections (Hero, Features, etc.)
│           │   └── ui/            # shadcn/ui component library
│           ├── hooks/
│           ├── lib/
│           └── ...config files
│
└── context.md                    # This file
```

### Key Design Decisions

- **No external runtime dependencies**: The interpreter (`NDJL/`) has zero npm dependencies — it uses only Node.js built-ins (`fs`, `path`).
- **Standalone distribution**: `pkg` bundles Node.js + the interpreter into a single `ndjl.exe` so end users do not need Node.js installed.
- **English-like syntax**: All control flow uses natural words (`greater than`, `less than`, `equals`, `repeat N times`, `ask ... into`).
- **Dual type aliases**: `int` and `number` are both numeric; `str` and `text` are both string — letting beginners use whichever feels natural.
- **Block delimiters**: All blocks (`if`, `repeat`, `define`) close with `end`, keeping structure explicit but lightweight.
- **Scoping**: Functions get a local scope that inherits from globals. There is no closure support; nested function definitions are not supported.
