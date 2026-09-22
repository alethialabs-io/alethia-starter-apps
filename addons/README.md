# `addons/`

Alethia renders a second ArgoCD Application, `addons`, pointed at this directory with
`recurse: true`. **It is rendered whether or not the directory exists**, so a repository without
one leaves a red `ComparisonError` Application in the ArgoCD UI after a deploy that reported
success. This directory existing — even holding nothing but this file — is what settles that.

ArgoCD reads `.yaml`, `.yml` and `.json` here. A Markdown file is ignored, so this README is not a
resource and the directory is empty as far as the Application is concerned.

## What goes here

Marketplace add-ons you enable in **GitOps mode**. Alethia writes `addons/<add-on>.yaml`, commits
it and pushes. The write is seed-once: an existing file is never overwritten, so your edits
survive, and Alethia removes only the files it authored — the ones carrying
`alethia.io/managed-by: addon-marketplace`.

## What you may put here yourself

The `addons` ArgoCD project is wider than the `apps` project in one specific way: its `sourceRepos`
accepts **any** repository, not only this one. That makes this the directory where an ArgoCD
`Application` pointing at an upstream chart belongs — see `alethia-starter-ai`, which installs
KServe and Kueue exactly this way.

Treat the contents as trusted content: the project permits cluster-scoped resources of any kind.
