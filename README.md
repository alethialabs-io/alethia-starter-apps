# alethia-starter-apps

A day-one **apps-destination repository** for [Alethia](https://alethialabs.io): the Git repository
an environment points ArgoCD at.

## What it is

Use this template, point an environment at your copy, deploy, and three Applications converge — the
root, `apps-dev` and `apps-staging`.

It is deliberately small. The value is not the workload; it is that every path in it is one the
product actually reads, with the reason written next to it.

```
kustomization.yaml     the ROOT manifest — without one the root Application manages nothing
namespace.yaml         the namespace the root delivers into
base/                  the application, once
overlays/dev/          discovered automatically; becomes the Application `apps-dev`
overlays/staging/      discovered automatically; becomes the Application `apps-staging`
addons/                keeps the `addons` Application out of ComparisonError
TEMPLATE_VERSION       semver; see CHANGELOG.md
```

## Use this template

**Use this template** (the green button) to create your own repository. You do not need to fork it,
and nothing here phones home.

## Connect it in Alethia

1. In Alethia, open the environment → **Repositories** → set **ArgoCD apps repository** to your new
   repository. Leave **Overlay path** empty.
2. Deploy.

Or from the CLI:

```bash
alethia project component add --project <project> --env <env> --kind repositories \
  --set apps_destination_repo=https://github.com/<you>/<your-repo>
```

A **public** repository needs no Git token — ArgoCD clones it anonymously. A private one needs the
Git provider connected for the job owner.

The docs page that covers this template, and the other two:
[Starter Templates](https://alethialabs.io/docs/console/design-project/starter-templates). What an
apps repository must contain, and what Alethia commits back into it:
[Repositories](https://alethialabs.io/docs/console/design-project/repositories).

### What you should see

| ArgoCD Application | Source path | Namespace | What it runs |
|---|---|---|---|
| `apps` | the repository root | `starter` | one `hello` Deployment, one Service |
| `apps-dev` | `overlays/dev` | `starter-dev` | the same, 1 replica, smallest requests |
| `apps-staging` | `overlays/staging` | `starter-staging` | the same, 2 replicas |
| `addons` | `addons/` | — | nothing yet, and Healthy |

```bash
kubectl -n starter-staging get deploy,svc
kubectl -n starter-staging port-forward svc/hello 8080:80
```

## The contract

### The four rules this template exists to demonstrate

**1. The root needs at least one manifest.** The root Application hardcodes `targetRevision: HEAD`
and `path: .` (or the overlay path you name). A root with nothing in it yields an Application that
is Synced and Healthy while managing zero resources — the usual reason a deploy succeeds and the
cluster stays empty. `kustomization.yaml` here is that manifest, and CI fails if the root ever
renders nothing.

**2. A root manifest opts you out of scaffolding.** Alethia generates service manifests into the
repository root and commits them — but only while the root holds no `.yaml` or `.yml` file. That
test looks at the root only, so a repository whose manifests all live under `overlays/` still reads
as empty and gets scaffolded files. One file at the root settles it for good.

**3. `overlays/*` is discovered, and only while no overlay path is named.** An environment that
names no path gets an `apps-overlays` ApplicationSet that globs `overlays/*` and renders one
Application per directory. Add `overlays/prod`, commit, and it appears — no Alethia-side change.
Naming an explicit overlay path turns discovery **off** for that environment, deliberately: the
glob is anchored at the repository root, so an environment that both named a path and discovered
would adopt the directories other environments are already delivering, and two self-healing
Applications would fight over the same resources.

**4. Each overlay ships its own Namespace.** The Applications Alethia renders name a server and no
destination namespace, so `CreateNamespace=true` has nothing to create, and a manifest pointing at a
namespace that does not exist fails to sync. Every directory here is therefore deliverable on its
own — which is what makes `apps_path=overlays/dev` work as well as discovery does.

### Which placement this template targets

**A dedicated cluster.** The root and the overlays each create a `Namespace`, which is a
cluster-scoped resource. The `apps` ArgoCD project is wide open on a dedicated cluster and permits
it; a **namespace placement** runs under a tenant project that allows no cluster-scoped resource at
all, and would refuse these manifests.

On a namespace or vcluster placement, drop the `Namespace` objects and the `namespace:`
transformers: Alethia has already created your namespace, the single Application it renders
delivers exactly one path, and overlay discovery does not run.

### A commit to the default branch deploys

The Applications sync automatically, prune, and self-heal:

- a commit to the default branch deploys on its own — no Alethia deploy, no approval step;
- deleting a manifest deletes the resource it created;
- editing a resource by hand puts it back at the next sync.

**Write access to the default branch is deploy access to the environment.** Restrict the branch if
that is not what you want.

### CI

`.github/workflows/validate.yml` runs on every push and costs nothing: `kustomize build` over the
root and every overlay, plus the four assertions above as explicit steps.

## Versioning

`TEMPLATE_VERSION` holds the current version, in [semantic versioning](https://semver.org).
`CHANGELOG.md` records each release and says which change is which bump. CI fails a change to the
delivered manifests that does not move `TEMPLATE_VERSION`. That version job runs only in
`alethialabs-io` — your clone's release discipline is yours.

## Licence

Apache-2.0. See [`LICENSE`](./LICENSE) and [`NOTICE`](./NOTICE).

## The other starter templates

| Repository | What it is |
|---|---|
| [`alethia-starter-apps`](https://github.com/alethialabs-io/alethia-starter-apps) | this one — the apps-destination repository |
| [`alethia-starter-chart`](https://github.com/alethialabs-io/alethia-starter-chart) | a bring-your-own Helm chart, under the default-deny project |
| [`alethia-starter-ai`](https://github.com/alethialabs-io/alethia-starter-ai) | RAG, a vector DB, CPU model serving and batch queueing, split across both |
| [`alethia-examples`](https://github.com/alethialabs-io/alethia-examples) | larger worked references, including the isolation ladder |
