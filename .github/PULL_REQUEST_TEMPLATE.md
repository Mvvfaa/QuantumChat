<!--
  Thank you for contributing to CalculusRuntime!

  Please fill in every section below. PRs without a clear description,
  linked issue, or completed checklist will be asked to resubmit.

  Tip: the title should follow Conventional Commits format:
    feat(slang): add fourier series solver
    fix(solver): handle division by zero in inference engine
    docs(slang): add chainRuleDifferentiate example
-->

## Summary

<!-- One or two sentences describing what this PR does and why. -->


## Related issue

<!--
  Every PR should be linked to an issue. Use one of:
    Closes #<issue-number>      — fully resolves the issue
    Fixes #<issue-number>       — fixes a bug reported in the issue
    Refs #<issue-number>        — related but does not close the issue

  If no issue exists for this change, briefly explain why one was not created.
-->

Closes #


## Type of change

<!-- Check all that apply. -->

- [ ] 🐛 Bug fix (non-breaking change that fixes an issue)
- [ ] ✨ New feature (non-breaking change that adds functionality)
- [ ] 💥 Breaking change (fix or feature that changes existing behaviour)
- [ ] ♻️ Refactor (no functional change)
- [ ] 📚 Documentation only
- [ ] 🧪 Tests only
- [ ] 🔧 CI / tooling / configuration
- [ ] ⬆️ Dependency update


## Affected component(s)

<!-- Check all that apply. -->

- [ ] SLaNg — symbolic engine (`src/core/`, `src/math/`, `src/utils/`)
- [ ] SLaNg — converter (`src/convertor.js`)
- [ ] SLaNg — SvelteKit demo website (`slang/website/`)
- [ ] CalculusSolver — data pipeline
- [ ] CalculusSolver — training (`model_trainer.py`)
- [ ] CalculusSolver — inference engine
- [ ] CalculusSolver — evaluation
- [ ] CalculusSolver — tokenizer
- [ ] CalculusSolver — React website (`calculussolver/website/`)
- [ ] Frontend (main React app)
- [ ] Backend (API)
- [ ] Setup scripts (`setup.sh` / `setup.ps1`)
- [ ] CI / GitHub Actions workflows
- [ ] Documentation


## What changed and why

<!--
  Explain the approach taken. For non-trivial changes, describe:
  - What the root cause was (for bug fixes)
  - Why this implementation was chosen over alternatives
  - Any trade-offs made
-->


## Testing

<!--
  Describe how you tested this change. Include:
  - What tests you ran (unit, integration, manual)
  - Any edge cases you specifically verified
  - Steps for a reviewer to reproduce your testing locally
-->

**Test commands run:**
```bash
# e.g.
npm test                     # SLaNg unit tests
python -m pytest             # CalculusSolver tests
npm run dev                  # manual UI check
```

**Edge cases verified:**
- [ ] Zero coefficients / empty inputs
- [ ] Division by zero / undefined expressions
- [ ] Single-term expressions
- [ ] Symbolic vs. numeric inputs (SLaNg)
- [ ] Large / deeply nested expressions


## Screenshots or output (if applicable)

<!--
  For UI changes: before/after screenshots.
  For SLaNg/solver changes: sample input → expected output → actual output.
  Delete this section if not applicable.
-->


## Documentation

<!--
  Tick whichever apply. Every new public SLaNg function needs a matching .md doc.
-->

- [ ] New or updated SLaNg API docs added to `docs/explaination/`
- [ ] `ARCHITECTURE.md` / `GUIDE.md` updated (if applicable)
- [ ] README updated (if applicable)
- [ ] No documentation changes needed

## Breaking changes

<!--
  If this PR introduces a breaking change, describe:
  - What breaks
  - Who is affected
  - What consumers need to do to migrate

  Delete this section if there are no breaking changes.
-->


## Checklist

<!-- All boxes must be checked before a reviewer will merge this PR. -->

- [ ] My branch is up to date with `upstream/main` and has no unresolved merge conflicts.
- [ ] My commit messages follow the [Conventional Commits](https://www.conventionalcommits.org/) format described in `CONTRIBUTING.md`.
- [ ] I have not committed secrets, API keys, or personal tokens.
- [ ] I have not committed `.pyc` files, `__pycache__/`, or `node_modules/`.
- [ ] All existing tests pass locally.
- [ ] I have added tests that cover my change (where applicable).
- [ ] I have read and agree to the [Code of Conduct](../CODE_OF_CONDUCT.md).
