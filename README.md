# Altertable Client Specs

Versioned API specifications for building Altertable open-source SDKs.

## Overview

This repository contains pure specifications: requirements, fixtures, constants, and test plans. SDK repositories consume it as a git submodule. Workspace skills read these specs and act on them.

SDK repositories pin a specific version tag of this repo via a `specs/` submodule, ensuring every SDK is built against a known, reproducible spec snapshot.

## Specs

| Spec | Description |
|---|---|
| [`lakehouse`](lakehouse/SPEC.md) | Production-grade Altertable Lakehouse API client — typed models, streaming, auth |
| [`product-analytics`](product-analytics/SPEC.md) | Product Analytics SDK — identity, event tracking, sessions, storage, consent, auto-capture |
| [`http`](http/SPEC.md) | HTTP transport requirements — connection pooling, keep-alive, timeouts, language-specific recommendations |

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

## Versioning

Spec versions follow [Semantic Versioning](https://semver.org). Each tag (e.g. `v0.1.0`) represents a stable, immutable snapshot. SDK repositories pin to a tag — never a branch — to guarantee reproducible builds.

## Workspace

Albert, the autonomous AI maintainer of these SDKs, operates from the [altertable-workspace](https://github.com/altertable-ai/altertable-workspace) repository. Operational skills (triage, review, release, sync) live there.

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
