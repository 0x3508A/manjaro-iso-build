# Manjaro ISO Build Repository
#### My Manjaro

## References:

- Building your custom Manjaro ISO via Github Actions CI
      - <https://www.youtube.com/watch?v=S2t5Iat37CI>
- Manjaro iso-profiles repository
      - <https://gitlab.manjaro.org/profiles-and-settings/iso-profiles>
      - Clone Link: `https://gitlab.manjaro.org/profiles-and-settings/iso-profiles.git`
- [CI] iso_build script *Original*
      - <https://gitlab.manjaro.org/-/snippets/630>
- Manjaro Packages Compare
      - <https://packages.manjaro.org/?query=%23manjaro>

## Modified CI Code

```yaml
name: iso_build
on:
  workflow_dispatch:
  # schedule:
  #  - cron:  '30 2 * * *'

jobs:
  prepare-release:
    runs-on: ubuntu-20.04
    steps:
      - 
        uses: styfle/cancel-workflow-action@0.9.0
        with:
          access_token: ${{ github.token }}
      - 
        id: time
        uses: nanzm/get-time-action@v1.1
        with:
          format: 'YYYYMMDDHHmm'
    outputs:
      release_tag: ${{ steps.time.outputs.time }}
  build-release:
    runs-on: ubuntu-20.04
    needs: [prepare-release]
    strategy:
      matrix:
        EDITION: [xfce]
        BRANCH: [stable]
        SCOPE: [minimal,full]
    steps:
      - 
        uses: styfle/cancel-workflow-action@0.9.0
        with:
          access_token: ${{ github.token }}
      - 
        id: time
        uses: nanzm/get-time-action@v1.1
        with:
          format: 'YY.MM'   
      -
        name: image-build-upload
        uses: manjaro/manjaro-iso-action@main
        with:
          edition: ${{ matrix.edition }}
          branch: ${{ matrix.branch }}
          scope: ${{ matrix.scope }}
          version: ${{ steps.time.outputs.time }}
          kernel: linux515
          code-name: "Custom-Build"
          release-tag: ${{ needs.prepare-release.outputs.release_tag }}
      -
        name: rollback
        if: ${{ failure() || cancelled() }}
        run: |
          echo ${{ github.token }} | gh auth login --with-token
          gh release delete ${{ needs.prepare-release.outputs.release_tag }} -y --repo ${{ github.repository }}

```
