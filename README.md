# Github Action to finish a gitflow release

This action fnishes a gitflow release.

# Usage

## Example

Here is an example of what to put in your `.github/workflows/finish-release.yml` file to use this workflow.

```yaml
name: Finish Release
on:
  pull_request:
    types:
      - closed
    branches:
      - master
env:
  git_user: some-bot                # CHANGE ME!
  git_email: some-bot@example.com   # CHANGE ME!
jobs:
  finish-release:
    if: github.event.pull_request.merged == true && (startsWith(github.head_ref, 'refs/heads/release/') || startsWith(github.head_ref, 'release/'))
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2
        with:
          ref: master                 # Always finish releases from the "merged to" master
          fetch-depth: 0
          persist-credentials: false  # Needed so we can push with different credentials.
                                      # NOTE: Pushing with different credentials allows admins to push protected branches.
                                      # NOTE: Pushing with different credentials allow workflows to trigger from the push.

      # FINISH THE RELEASE
      - name: Initialize mandatory git config
        run: |
          git config --global user.name $git_user &&
          git config --global user.email $git_email
      - name: Finish gitflow release
        id: finish-release
        uses: duplocloud/ghactions-finish-gitflow-release@master
        with:
          github_token: ${{ secrets.RELEASE_BOT_GITHUB_TOKEN }}
```

## Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| github_token | Github token to use for pushing the tag and deleting the release branch | true | unset |
| target_ref | release target ref (pull request base ref - defaults to `${{ github.base_ref }}`, which is usually master) | false | `""` |
| source_ref | release source ref (pull request head ref - defaults to `${{ github.head_ref }})`' | false | `""` |
| develop_ref | develop ref (defaults to develop) | false | `"develop"` |
| tag_prefix | The tag prefix (defaults to "v") | false | `"v"` |
| delete_branch | Whether or not to delete the release branch. Set this to `false` if your source ref has previously been deleted. | false | `true` |
| validate_merge | Validates that the `source_ref` was already merged into the `target_ref`.  Set this to `false` if you merge a squashed commit or if your source ref has previously been deleted. | false | `true` |

## Outputs

| Name | Description |
|------|-------------|
| release_tag | release git tag (with version prefix) |
| release_branch | release git branch |
| release_commit | release git commit |
| source_ref | release source ref (pull request head) |
| target_ref | release target ref (pull request base) |
