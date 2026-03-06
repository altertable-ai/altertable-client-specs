# Altertable Client Specs

Versioned API specifications and agent skills for building and maintaining Altertable open-source SDKs.

## Overview

This repository serves two purposes:

1. **Client specs** — OpenAPI specifications and shared constants consumed by Altertable SDK repositories as a git submodule.
2. **Agent skills** — Instruction sets for AI agents that implement, maintain, and release SDKs against those specs.

SDK repositories pin a specific version tag of this repo via a `specs/` submodule, ensuring every SDK is built against a known, reproducible spec snapshot.

## Skills

| Skill                                                                        | Description                                                                                           |
| ---------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| [`bootstrap-sdk`](skills/bootstrap-sdk/SKILL.md)                             | Fork, clone, and wire up a new SDK repo or update an existing one to a new spec version               |
| [`build-lakehouse-sdk`](skills/build-lakehouse-sdk/SKILL.md)                 | Build a production-grade Altertable Lakehouse API client in any language                              |
| [`build-product-analytics-sdk`](skills/build-product-analytics-sdk/SKILL.md) | Build an Altertable Product Analytics SDK with identity, event tracking, and auto-capture             |
| [`build-http-sdk`](skills/build-http-sdk/SKILL.md)                           | HTTP client best practices — connection pooling, keep-alive, timeouts (referenced by build-\* skills) |
| [`build-readme`](skills/build-readme/SKILL.md)                               | Write READMEs for SDK repos and monorepo roots following Altertable conventions                       |
| [`maintainer-routine`](skills/maintainer-routine/SKILL.md)                   | Notification-driven maintainer routine to identify actionable work across Altertable SDK repositories |
| [`release-sdk`](skills/release-sdk/SKILL.md)                                 | Release SDKs, write changelogs, and publish to language registries                                    |
| [`review-pr`](skills/review-pr/SKILL.md)                                     | Review community pull requests against Altertable SDK standards                                       |
| [`sync-repos`](skills/sync-repos/SKILL.md)                                   | Keep shared configuration, community files, and CI templates consistent across SDK repositories       |
| [`triage-issues`](skills/triage-issues/SKILL.md)                             | Triage incoming GitHub issues across Altertable SDK repositories                                      |

## Using This Repo as a Submodule

To consume a pinned version of these specs in an SDK repository:

```bash
git submodule add https://github.com/altertable-ai/altertable-client-specs.git specs
git -C specs checkout <spec-tag>
git add .gitmodules specs
git commit -m "chore: add altertable-client-specs submodule at <spec-tag>"
```

After cloning an SDK repo that already includes this submodule:

```bash
git submodule update --init --recursive
```

See the [`bootstrap-sdk`](skills/bootstrap-sdk/SKILL.md) skill for the full workflow, including fork strategy, branch naming, and PR conventions.

## Versioning

Spec versions follow [Semantic Versioning](https://semver.org). Each tag (e.g. `v0.1.0`) represents a stable, immutable snapshot. SDK repositories pin to a tag — never a branch — to guarantee reproducible builds.

## Maintainer Agent

This repository ships a [`SOUL.md`](SOUL.md) that defines **Albert**, the autonomous AI maintainer responsible for triaging issues, reviewing PRs, and keeping the SDK ecosystem healthy. Albert operates via fork + PR.

## Contributing

1. Fork this repository
2. Create a branch: `feat/<short-desc>` or `fix/<issue-number>-<short-desc>`
3. Commit your changes with a clear message
4. Push to your fork
5. Open a pull request against `main`

## License

[MIT](LICENSE)

## Links

- Website: [https://altertable.ai](https://altertable.ai)
- Documentation: [https://altertable.ai/docs](https://altertable.ai/docs)
- GitHub: [https://github.com/altertable-ai/altertable-client-specs](https://github.com/altertable-ai/altertable-client-specs)
