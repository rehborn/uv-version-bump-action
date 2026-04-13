# uv version bump action

unofficial action to [update your version](https://docs.astral.sh/uv/guides/package/#updating-your-version)
using [astral-sh/uv](https://github.com/astral-sh/uv)
and create [git tag](https://git-scm.com/book/en/v2/Git-Basics-Tagging) according to
[PEP440](https://peps.python.org/pep-0440/)

- bump version with uv (update pyproject.toml+uv.lock) and push to origin
- create a git tag for new version and push to remote
- return `VERSION` for further usage

### inputs

| argument         | default                                        | description                                                                         |
|------------------|------------------------------------------------|-------------------------------------------------------------------------------------|
| `bump`           | `minor`                                        | Version Bump (major, minor, patch + alpha, beta, rc, post, and dev, ie "minor dev") |
| `github_token`   | `${{ github.token }}`                          | GITHUB_TOKEN for updating pyproject.toml and uv.lock and pushing version as tag     |
| `commit_message` | `bump to version`                              | git commit message                                                                  |
| `commit_user`    | `github-actions[bot]`                          | git commit user                                                                     |
| `commit_email`   | `github-actions[bot]@users.noreply.github.com` | git commit email                                                                    |
| `remote_host`    | `github.com`                                   | git remote host                                                                     |
| `tag_message`    | `version`                                      | git tag message                                                                     |
| `tag_prefix`     | *empty*                                        | prefix for git tag                                                                  | 

### minimal example

```yaml
  - name: Version Bump
    uses: rehborn/uv-version-bump-action@v0.0.2
    with:
      bump: "major dev"
      github_token: ${{ secrets.GITHUB_TOKEN }}
```

### full example

```yaml
  - name: Version Bump
    uses: rehborn/uv-version-bump-action@v0.0.2
    with:
      bump: "major dev"
      github_token: ${{ secrets.GITHUB_TOKEN }}
      commit_message: 'Bump Release Version'
      tag_message: 'Release'
```

### custom commiter and host example
compatible with forgejo/gitea runner
```yaml
      - name: Version Bump
        id: version
        uses: https://github.com/rehborn/uv-version-bump-action@v0.0.3
        with:
          commit_user: 'bot'
          commit_email: 'bot@boxnet.eu'
          remote_host: 'git.boxnet.eu'

          bump: ${{ inputs.bump }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

### complete workflow example

```yaml
name: Bump Version

on:
  workflow_dispatch:
    inputs:
      bump:
        required: false
        description: Version Bump (major, minor, patch + alpha, beta, rc, post, and dev)
        type: string
        default: 'patch'

jobs:
  bump:
    name: Version Bump
    runs-on: ubuntu-latest
    permissions:
      contents: write

    # store for use with reusable actions
    outputs:
      VERSION: "${{ steps.version.outputs.VERSION }}"

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          persist-credentials: false

      - name: Version Bump
        id: version
        uses: rehborn/uv-version-bump-action@v0.0.3
        with:
          bump: ${{ inputs.bump }}
          github_token: ${{ secrets.GITHUB_TOKEN }}

  # Access VERSION in job
  output:
    needs: bump
    runs-on: ubuntu-latest
    name: test output
    steps:
      - name: test
        run: echo ${{ needs.bump.outputs.VERSION }}

  # Pass VERSION to Jobs
  build:
    needs: bump
    permissions:
      packages: write
      contents: write
      id-token: write
    uses: ./.github/workflows/build.yml
    secrets: inherit
    with:
      version: ${{ needs.bump.outputs.VERSION }}

  release:
    needs:
      - bump
      - build
    permissions:
      packages: write
      contents: write
    uses: ./.github/workflows/release.yml
    secrets: inherit
    with:
      version: ${{ needs.bump.outputs.VERSION }}
```
