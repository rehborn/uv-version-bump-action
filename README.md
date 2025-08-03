# uv version bump action

- bump version with uv (update pyproject.toml+uv.lock) and push to main
- create a git tag for new version and push to remote
- return VERSION for further usage

### inputs
|argument||
|---|---|
|`bump`|Version Bump (major, minor, patch + alpha, beta, rc, post, and dev, ie "minor dev")|
|`github_token`|github token for updating pyproject.toml and uv.lock and pushing version as tag|

### minimal example
```yaml
  - name: Version Bump
    uses: rehborn/uv-version-bump-action@v0.0.1
    with:
      bump: "major dev"
      github_token: ${{ secrets.GITHUB_TOKEN }}
```


### full example
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
        uses: rehborn/uv-version-bump-action@0.0.1
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

  # Pass VERSION to Workflow
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
```