# Changelog

This template is versioned with [semantic versioning](https://semver.org). `TEMPLATE_VERSION` at
the repository root holds the current version, and CI fails a change to the delivered manifests
that does not move it.

What counts as which, for this repository:

| Change | Bump |
|---|---|
| A layout change a clone must follow — a moved or renamed path, a resource that must be deleted by hand | **major** |
| A new overlay, a new manifest, a pinned image version | **minor** |
| A comment, a README, a CI tweak | **patch** |

## 1.0.0 — 2026-09-22

Initial template.

- Root Kustomization plus a `starter` Namespace, so the root `apps` Application manages at least
  one resource and the repository is opted out of generated-manifest scaffolding.
- `base/` holding the application once: one Deployment, one Service.
- `overlays/dev` and `overlays/staging`, each shipping its own Namespace so it is deliverable on
  its own under an explicit `apps_path` as well as through overlay discovery.
- `addons/`, so the `addons` Application does not report a ComparisonError.
- CI: `kustomize build` over the root and every overlay, on every push.
