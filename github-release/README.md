# github-release

A GitHub Action which creates a [GitHub release](https://docs.github.com/en/repositories/releasing-projects-on-github/about-releases).

```yaml
github-release:
  runs-on: ubuntu-latest
  if: ${{ !github.event.repository.fork && github.ref == 'refs/heads/main' }}
  needs: [all-required-checks-complete]
  environment: main-deploy
  permissions:
    contents: write
  steps:
    - name: Compute tag
      id: compute-tag
      # Do something to compute the tag here, e.g. "ApiSurface_1.2.3".
      # Optionally also push it to the repo; or alternatively just let the `github-release` step do the tag creation.
    - name: Create GitHub release
      uses: G-Research/common-actions/github-release@main
      with:
        tag: ${{ steps.compute-tag.output }}
        target-commitish: ${{ github.sha }}
        github-token: ${{ secrets.GITHUB_TOKEN }}
        # Optionally:
        generate-release-notes: true
        draft: true
        prerelease: true
        binary-contents: |
          path/to/binary1
          path/to/binary2
```

# Why?

GitHub releasing is an operation which is intended to happen in a privileged context.
It's very simple to do using the GitHub API, but GitHub appear not to offer a first-party action to do it.
We prefer to keep our dependency footprints small in privileged contexts; so we simply make the necessary API calls manually.

# Inputs

## `tag`

The tag name to which the GitHub release will correspond.
This is used both as the release name and as the tag to which the release points.

If this doesn't exist at the time this step runs, the tag is created to point to `target-commitish`.

## `github-token`

A GitHub token with at least `contents: "write"` perms; also optionall `actions: "write"` (GitHub does not appear to document the conditions under which this is required; we believe it's when you want this step to run correctly on pull requests which edit a `.github/` workflow file).
This should usually be `github-token: ${{ secrets.GITHUB_TOKEN }}`, and you need to remember the appropriate `permissions` block in the job config.

## `target-commitish`

Specifies the commitish value that will be the target for the release's tag.
Required if the supplied `tag` does not reference an existing tag, and also required if `generate-release-notes` is true (though GitHub does not document this).

If you don't specify this, the current head of the repo's default branch is used; it seems likely that this results in race conditions when multiple instances of the pipeline run simultaneously.
`${{ github.sha }}` seems like a sensible value to use when the workflow is running on the default branch.

## `draft`

Boolean.
Set to `true` to mark this GitHub release as a draft.

## `prerelease`

Boolean.
Set to `true` to mark this GitHub release as a prerelease.

## `generate-release-notes`

Boolean.
Enables [generated release notes](https://docs.github.com/en/repositories/releasing-projects-on-github/automatically-generated-release-notes) on the GitHub release.
If you set this, you must also set `target-commitish` (even if the `tag` already exists), though GitHub does not appear to document this fact.

## `binary-contents`

Paths to some binary data to upload to the release as release assets.

The simpler (but less flexible) way to pass inputs here is as a single string (if you want to upload only one asset):

```yaml
with:
  binary-contents: foo
```

or as a newline-delimited string list:

```yaml
with:
  binary-contents: |
    foo
    bar
    baz
```

However, you may instead use the canonical input format, a string containing a JSON array of filepaths.
(You *must* use this format if any of your paths contain newlines, or if they are themselves certain kinds of valid JSON string.)
This eccentric input method is because [GitHub Actions doesn't support lists](https://github.com/actions/toolkit/issues/184).

```yaml
with:
  binary-contents: "[\"hello\nworld.txt\",\"foo.txt\"]"
```

(We wish you the best of luck constructing this string. YAML is hard for humans to read or write; use [an interactive renderer](https://yaml-online-parser.appspot.com/).)
