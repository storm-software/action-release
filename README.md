# Storm Software - Release

Composite GitHub Action that derives Nx affected SHAs, warms up the devenv
shell (with retries for intermittent store-path flake failures), and runs the
repository `release` command.

Use after [`storm-software/action-devenv-setup`](https://github.com/storm-software/action-devenv-setup).

## Usage

```yaml
- name: Setup workspace
  uses: storm-software/action-devenv-setup@main
  with:
    gpg-sign-key: ${{ secrets.STORM_BOT_GPG_SIGN_KEY }}
    gpg-private-key: ${{ secrets.STORM_BOT_GPG_PRIVATE_KEY }}
    gpg-passphrase: ${{ secrets.STORM_BOT_GPG_PASSPHRASE }}
    storm-bot-github-token: ${{ secrets.STORM_BOT_GITHUB_TOKEN }}
    cachix-auth-token: ${{ secrets.CACHIX_AUTH_TOKEN }}

- name: Release repository updates
  uses: storm-software/action-release@main
  with:
    workflow-id: release.yml
    main-branch-name: main
    devenv-profile: production
    tag: ${{ inputs.tag }}
    env: |
      {
        "GITHUB_ACTOR": "${{ github.actor }}",
        "GITHUB_REPOSITORY": "${{ github.repository }}",
        "GITHUB_TOKEN": "${{ github.token }}",
        "STORM_BOT_GITHUB_TOKEN": "${{ secrets.STORM_BOT_GITHUB_TOKEN }}",
        "STORM_WORKSPACE_ROOT": "${{ github.workspace }}",
        "STORM_REPOSITORY": "${{ github.repositoryUrl }}",
        "NPM_TOKEN": "${{ secrets.STORM_BOT_NPM_TOKEN }}",
        "CLOUDFLARE_API_TOKEN": "${{ secrets.STORM_BOT_CLOUDFLARE_TOKEN }}"
      }
```

### Custom environment variables

Pass per-repo secrets and vars in either (or both) of these ways:

1. **`env` input (JSON)** — applied only to the release step. Prefer this for
   release-only credentials that differ across repositories.
2. **Step-level `env:` on `uses:`** — inherited by all composite steps
   (SHA derivation, warm-up, and release).

```yaml
- uses: storm-software/action-release@main
  with:
    env: |
      {
        "NPM_TOKEN": "${{ secrets.STORM_BOT_NPM_TOKEN }}",
        "CARGO_REGISTRY_TOKEN": "${{ secrets.STORM_BOT_CARGO_TOKEN }}"
      }
  env:
    CI: true
```

`NX_BASE`, `NX_HEAD`, and `TAG` are always set on the release step from the
derived SHAs and the `tag` input.

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `workflow-id` | `release.yml` | Workflow file for last successful run lookup |
| `main-branch-name` | `main` | Main branch for Nx affected base |
| `error-on-no-successful-workflow` | `false` | Fail when no successful workflow is found |
| `devenv-profile` | `production` | Profile for `devenv --profile` |
| `tag` | _(empty)_ | Optional tag override (`TAG`) |
| `release-args` | _(empty)_ | Extra args after base/head SHAs |
| `env` | `{}` | JSON object of release-step environment variables |

## Outputs

| Output | Description |
| --- | --- |
| `base` | Base SHA for nx affected / release |
| `head` | Head SHA for nx affected / release |

## License

Apache-2.0
