---
name: bootstrap-sdk
description: Bootstrap or update an Altertable SDK repository from a versioned client spec. Use when given a target GitHub repository (e.g. altertable-ai/altertable-lakehouse-ruby) and a spec version tag (e.g. v0.1.0) to set up a git submodule, implement missing functionality, and submit a PR. Also use when a new spec version is released and the SDK needs to be updated to match.
---

# Bootstrap SDK from Client Specs

## Purpose

Use this skill to initialize or update an SDK repository against a specific version of `altertable-ai/altertable-client-specs`. All contributions go through a fork + branch + PR workflow — the maintainer agent has no direct write access to the target repository.

## Related Skills

This skill coordinates with other SDK development skills:

- **[build-lakehouse-sdk](../build-lakehouse-sdk/SKILL.md)**: Implementation guide for Lakehouse API clients
- **[build-product-analytics-sdk](../build-product-analytics-sdk/SKILL.md)**: Implementation guide for Product Analytics SDKs
- **[build-http-sdk](../build-http-sdk/SKILL.md)**: HTTP client best practices (referenced by build-* skills)
- **[release-sdk](../release-sdk/SKILL.md)**: Versioning, changelog, and registry publishing conventions

## Inputs

Collect before starting:

- **Target repo**: GitHub repository slug (e.g. `altertable-ai/altertable-lakehouse-ruby`)
- **Spec tag**: Tag of `altertable-ai/altertable-client-specs` to target (e.g. `v0.1.0`)
- **SDK type**: Which SDK skill applies (`lakehouse-sdk`, `product-analytics-sdk`, etc.)

## Workflow

### Phase 1: Fork and clone

1. Fork the target repo to your GitHub account (skip if fork already exists)
2. Clone your fork locally
3. Add the upstream remote
4. Create a branch. For initial bootstrap use `bootstrap/specs-<spec-tag>`; for spec updates use `update/specs-<spec-tag>`

### Phase 2: Set up the specs submodule

**Initial bootstrap** (submodule does not exist yet):

```bash
git submodule add https://github.com/altertable-ai/altertable-client-specs.git specs
git -C specs checkout <spec-tag>
```

Then pin the submodule to the exact tag commit and commit:

```bash
git add .gitmodules specs
git commit -m "chore: add altertable-client-specs submodule at <spec-tag>"
```

**Spec update** (submodule already exists):

1. Identify the previous spec tag by reading `.gitmodules` and checking the submodule's current HEAD.
2. Update to the new tag:

```bash
git -C specs fetch --tags
git -C specs checkout <new-spec-tag>
```

3. Inspect the diff to understand what changed:

```bash
git -C specs diff <old-tag>..<new-spec-tag> -- .
```

4. Stage and commit the submodule pointer update:

```bash
git add specs
git commit -m "chore: update altertable-client-specs submodule to <new-spec-tag>"
```

### Phase 3: Implement or update the SDK

Read the specs submodule to understand the API surface, then apply the appropriate SDK skill:

- For Lakehouse SDKs: read and follow [build-lakehouse-sdk](../build-lakehouse-sdk/SKILL.md)
- For Product Analytics SDKs: read and follow [build-product-analytics-sdk](../build-product-analytics-sdk/SKILL.md)

**Initial bootstrap**: implement everything required by the skill from scratch.

**Spec update**: use the spec diff from Phase 2 to identify what changed. Only implement what is new or modified. Document breaking changes in `CHANGELOG.md`.

### Phase 4: Validate

Before opening the PR:

- [ ] All tests pass (`lint`, `typecheck`, `unit`, `integration` where applicable)
- [ ] Submodule points to the correct spec tag commit
- [ ] `CHANGELOG.md` is updated with a new entry
- [ ] `README.md` reflects any new public API surface
- [ ] Package version bumped if applicable (follow [release-sdk](../release-sdk/SKILL.md) conventions)

### Phase 5: Open a PR

Push the branch to your fork and open a PR against the upstream `main` branch.

## Decision tree

```
Given: target repo + spec tag
│
├── Does the repo have a `specs` submodule?
│   ├── No  → Phase 1 (fork/clone) → Phase 2 (initial submodule) → Phase 3 (full implementation)
│   └── Yes → Phase 1 (fork/clone) → Phase 2 (update submodule) → Phase 3 (diff-based update)
│
└── Always → Phase 4 (validate) → Phase 5 (open PR)
```

## Notes

- Never push directly to the target repo's `main` branch — always go through fork + PR.
- The `specs/` directory should be treated as read-only; never modify files inside it.
- Pin the submodule to the tag's commit SHA, not a branch, to ensure reproducibility.
- If the fork already exists and is stale, sync it before branching: `gh repo sync <your-fork> --source <target-repo>`.
- **Monorepo exception**: Product Analytics web framework wrappers (React, Vue, Svelte, etc.) live in the [`altertable-js` monorepo](https://github.com/altertable-ai/altertable-js) under `packages/`, not in separate repositories. For these, skip the fork/clone workflow and work directly in the monorepo.
