# Contributing to FeynRules v2.3

Thank you for helping maintain this FeynRules source tree. The checked-in
package identifies itself as FeynRules 2.3.49 (29 September 2021). Changes
should preserve established physics results and model semantics while fixing
focused defects or improving compatibility with current Wolfram kernels and
external tools.

This is a legacy Wolfram Language codebase. Small, reviewable changes with an
explicit regression example are safer than broad cleanup or reformatting.

## Repository Layout

| Path | Purpose |
|------|---------|
| `FeynRules.m` | Entry point, compatibility setup, and parallel-kernel loading |
| `FeynRulesPackage.m` | Public symbols, package initialization, and component loading |
| `Core/` | Model declarations, index algebra, vertex extraction, decays, loops, and related symbolic routines |
| `Interfaces/` | FeynArts, CalcHEP, MadGraph, Sherpa, UFO, WHIZARD, TeX, ASperGe, and SuSpect interfaces and templates |
| `Models/` | Bundled examples, the Standard Model, restrictions, and the model template |
| `NLOCT.m` | NLO counterterm functionality |
| `ToolBox.m` | Shared user-facing and internal helper functions |
| `FRPalette.nb` | Mathematica palette |
| `UpdateNotes.txt` | Historical FeynRules 2.3 release notes |

## Before Making a Change

- Reproduce a bug in a fresh kernel and reduce it to the smallest relevant
  model and Wolfram Language expression.
- Record `$Version`, `$SystemID`, and `FR$VersionNumber`.
- Decide whether the change belongs to the symbolic core, package loader, a
  model, or one interface. Avoid unrelated cleanup in the same patch.
- Search all package files before renaming a symbol. Components are loaded
  eagerly from `FeynRulesPackage.m`, and names may also occur in model or
  interface templates.
- For a physics change, state the convention being used and provide a
  reference or an independently checkable derivation.

## Loading the Development Checkout

A Wolfram Engine or Mathematica installation is required for the core package.
External generators, compilers, and libraries are needed only when testing the
interfaces that use them.

Start a fresh kernel in the repository root and evaluate:

```wl
$FeynRulesPath = Directory[];
FR$Parallel = False;
Get[FileNameJoin[{$FeynRulesPath, "FeynRules.m"}]];
```

Setting `FR$Parallel = False` makes initial debugging deterministic and avoids
loading a second copy in parallel kernels. Do not rely on an installed copy of
FeynRules while testing a checkout. Confirm the loaded source with:

```wl
{$FeynRulesPath, $Version, $SystemID, FeynRules`FR$VersionNumber}
```

Use a new kernel after editing package code. `FR$Loaded` prevents an already
loaded package from being loaded again, and FeynRules maintains substantial
global state after a model has been read.

## Coding Guidelines

### Wolfram Language source

- Follow the local style in the file being changed. Do not mass-format legacy
  `.m` or `.fr` files as part of a functional fix.
- Keep patterns as narrow as the intended mathematical domain. Preserve
  evaluation order, conditions (`/;`), attributes, and the exact shape of
  returned expressions unless the change deliberately modifies them.
- Give public symbols a `::usage` message in the ``FeynRules` `` context. Put
  implementation-only symbols in the existing ``PRIVATE` `` context or use a
  package-specific prefix already established by the surrounding code.
- Avoid accidental collisions with symbols added by newer Wolfram releases.
  Qualify public names explicitly when a context is ambiguous, and use a
  package-owned name for internal helpers instead of changing a ``System` ``
  symbol.
- Treat `M$*`, `MR$*`, `FR$*`, and other global state as shared package state.
  Initialize or clear it consistently in serial and parallel kernels.
- If a new source file is added, include it in the appropriate loading block
  in `FeynRulesPackage.m` and verify that its context is correct.
- Add or update messages for user-visible failure modes. Do not replace a
  diagnosable failure with silent fallback behavior.

### Models and physics content

- Keep model declarations in `.fr` files. Update `M$Information` when authors,
  references, or model provenance change.
- State normalization, sign, index, flavor, gauge, and complex-conjugation
  conventions when they are not already fixed by the surrounding model.
- Check that a change preserves quantum numbers, Hermiticity, kinetic-term
  normalization, the mass spectrum, and representative vertices wherever
  those checks apply.
- Do not change a bundled reference model merely to hide a core regression.
  Fix the owning implementation or explain why the model itself is wrong.

### Interfaces and generated files

- Modify the source interface or checked-in template, not generated output.
- Keep compatibility with the language level and naming conventions already
  used by that backend's C, C++, Fortran, Python, or Wolfram Language files.
- Generate into a temporary directory. Do not commit build products, generated
  model directories, logs, caches, or parameter cards unless they are an
  intentional review fixture.
- When an external tool is required, report its exact version and distinguish
  generation success from compilation or execution success.

### Notebooks and file metadata

- Put reusable functionality in text-based source files rather than only in a
  notebook.
- For `.nb` changes, clear irrelevant outputs and avoid front-end metadata
  churn. Review the textual diff before committing.
- Preserve existing line endings and executable bits. Use
  `git diff --summary` to catch accidental mode changes.
- Do not update `FR$VersionNumber`, `FR$VersionDate`, or `UpdateNotes.txt` in an
  ordinary patch unless the change is explicitly part of a release.

## Testing

This repository currently has no automated test suite or CI configuration.
Every pull request must therefore include a reproducible Wolfram Language
regression and the observed result, not only a statement that the package
loaded.

### Baseline smoke test

After loading the checkout as shown above, evaluate:

```wl
SetDirectory[
  FileNameJoin[{$FeynRulesPath, "Models", "FirstExample"}]
];
FeynRules`LoadModel["FirstExample.fr"];

And[
  TrueQ[Global`FR$Loaded],
  SameQ[FeynRules`M$ModelName, "First_Example"],
  MemberQ[FeynRules`MR$ClassesList, FeynRules`F[1]],
  MemberQ[FeynRules`MR$ClassesList, FeynRules`V[1]]
]
```

The final expression must return `True`. This is a loading smoke test only; it
does not validate a symbolic or physics change. Explicit contexts keep the
same snippet reliable in a headless script that is parsed before FeynRules is
loaded. The model loader also appends internal field classes, so do not assert
that `MR$ClassesList` contains exactly the two classes declared by the example.

### Change-specific regression

Choose the smallest test that exercises the changed behavior:

| Change | Minimum evidence |
|--------|------------------|
| Loader, contexts, or public API | Load in a fresh kernel; show symbol contexts, messages, and representative calls |
| Core algebra or vertex extraction | Load the smallest relevant model and compare the exact affected expression or vertex list |
| Model definition | Run applicable `CheckHermiticity`, `CheckMassSpectrum`, and `CheckKineticTermNormalisation` checks, plus representative `FeynmanRules` calls |
| Interface | Generate the backend output in a clean temporary directory and inspect or run it with the target tool |
| Parallel code | Run the regression first with `FR$Parallel = False`, then with parallel kernels enabled |
| Palette or notebook | Test in the Mathematica front end and include a concise description or screenshot of the result |

For a compatibility fix, reproduce the failure on the affected Wolfram
version and, when available, rerun the regression on one previously working
version.

Use the bundled `Models/FirstExample/FirstExample.fr` for fast core smoke tests
and `Models/SM/SM.fr` only when Standard Model structure is relevant. A slow,
broad calculation is not a substitute for a small assertion that would fail
before the patch.

Before submitting, also run:

```sh
git diff --check
git diff --summary
git status --short
```

If a required Wolfram version or external backend is unavailable, say exactly
which part was not run. Do not describe generated output as validated by an
external tool unless that tool was actually executed.

## Reporting Bugs

A useful bug report includes:

- the exact FeynRules revision and whether another FeynRules installation is
  present on `$Path`;
- `$Version`, `$SystemID`, and whether parallel loading was enabled;
- a minimal model or a reference to one of the bundled models;
- the complete input needed to reproduce the problem in a fresh kernel;
- the full messages and relevant output;
- the expected result and the physics or software reason for expecting it; and
- external-tool versions for interface failures.

Remove private paths, unpublished model data, and credentials before posting a
notebook or log.

## Pull Requests

Keep each pull request centered on one defect or feature. The description
should include:

- the problem and its user-visible or physics impact;
- why the chosen layer owns the fix;
- any public-symbol, model-format, or generated-interface compatibility impact;
- the exact regression input and before/after result;
- the Wolfram and external-tool versions tested; and
- any untested platform, backend, or expensive calculation.

Review the complete diff, including notebook metadata and file modes. Do not
include generated artifacts or a version bump unless they are part of the
reviewed change.

## Commit Message Convention

Write commit messages in English using:

```text
<scope>(<target>): <subject>

<body>

<footer>
```

Use one primary scope. The target is a lowercase, kebab-case subsystem or
backend name without a path prefix or file extension.

| Scope | Use for | Target examples |
|-------|---------|-----------------|
| `core` | Symbolic and physics routines in `Core/` or `ToolBox.m` | `vertices`, `indices`, `decay`, `toolbox` |
| `package` | Entry points, public API, initialization, and loading | `loader`, `api`, `parallel` |
| `interface` | Exporters and external-tool templates | `ufo`, `madgraph`, `calchep`, `feynarts`, `sherpa`, `whizard`, `tex`, `asperge` |
| `model` | Bundled models, restrictions, examples, and templates | `sm`, `first-example`, `template` |
| `nlo` | NLOCT and bundled NLO data | `nloct`, `smqcd` |
| `compat` | Wolfram-version or platform compatibility | `symbols`, `parallel-kernels`, `macos` |
| `ui` | Palette and notebook-only user interface changes | `palette` |
| `docs` | Contributor and release documentation | `contributing`, `update-notes` |
| `repo` | Repository-wide maintenance | `gitignore`, `ci`, `release` |

### Subject and body

- Keep the full subject line at most 72 characters and the subject text after
  the colon at most 50 characters.
- Use imperative mood: `fix`, not `fixed` or `fixes`.
- Start the subject with a lowercase letter and do not end it with a period.
- Separate the body with one blank line and wrap it at 72 characters.
- Explain why the change is needed and note important compatibility or physics
  consequences. Do not merely repeat the diff.

Examples:

```text
core(vertices): preserve fermion operator ordering
```

```text
interface(ufo): fix form-factor serialization
```

```text
compat(symbols): avoid System name collisions

- Keep package-owned symbols in the FeynRules context
- Preserve vertex extraction on newer Wolfram kernels

Assisted-by: Codex:gpt-5.6-sol
```

## AI Attribution

Following the
[Linux kernel guidance for AI coding assistants](https://docs.kernel.org/process/coding-assistants.html),
commits that contain AI-assisted changes must include one trailer per
assistant:

```text
Assisted-by: AGENT_NAME:MODEL_NAME
```

The human committer remains responsible for reviewing and testing the change.
Do not use `Co-authored-by` for an AI assistant, and never add a
`Signed-off-by` line on behalf of a human contributor.

Use these canonical agent names:

| Agent name | Tool |
|------------|------|
| `ClaudeCode` | Anthropic Claude Code |
| `GitHub-Copilot` | GitHub Copilot |
| `OpenCode` | OpenCode CLI |
| `Codex` | OpenAI Codex |

Write the model name in lowercase and include a version or variant when known,
for example `claude-opus-4.6` or `gpt-5.6-sol`.
