# Github Packages
## [Publishing images to GitHub Packages](https://docs.github.com/en/actions/guides/publishing-docker-images#publishing-images-to-github-packages)

Each time you create a new release on GitHub, you can trigger a workflow to publish your image. The workflow in the example below runs when the `release` event triggers with the `created` activity type. For more information on the `release` event, see "[Events that trigger workflows](https://docs.github.com/en/actions/reference/events-that-trigger-workflows#release).

In the example workflow below, we use the Docker `build-push-action` action to build the Docker image, and if the build succeeds, push the built image to GitHub Packages.

The `build-push-action` options required for GitHub Packages are:

-   `username`: You can use the `${{ github.actor }}` context to automatically use the username of the user that triggered the workflow run. For more information, see "[Context and expression syntax for GitHub Actions](https://docs.github.com/en/actions/reference/context-and-expression-syntax-for-github-actions#github-context)."
-   `password`: You can use the automatically-generated `GITHUB_TOKEN` secret for the password. For more information, see "[Authenticating with the GITHUB\_TOKEN](https://docs.github.com/en/actions/automating-your-workflow-with-github-actions/authenticating-with-the-github_token)."
-   `registry`: Must be set to `docker.pkg.github.com`.
-   `repository`: Must be set in the format `OWNER/REPOSITORY/IMAGE_NAME`. For example, for an image named `octo-image` stored on GitHub at `http://github.com/octo-org/octo-repo`, the `repository` option should be set to `octo-org/octo-repo/octo-image`.

YAML

```yaml
name: Publish Docker image
on:
  release:
    types: [published]
jobs:
  push_to_registry:
    name: Push Docker image to GitHub Packages
    runs-on: ubuntu-latest
    permissions:
      packages: write
      contents: read
    steps:
      - name: Check out the repo
        uses: actions/checkout@v2
      - name: Push to GitHub Packages
        uses: docker/build-push-action@v1
        with:
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
          registry: docker.pkg.github.com
          repository: my-org/my-repo/my-image
          tag_with_ref: true
```

The above workflow checks out the GitHub repository, and uses the `build-push-action` action to build and push the Docker image. It sets the `build-push-action` option [`tag_with_ref`](https://github.com/marketplace/actions/build-and-push-docker-images#tag_with_ref) to automatically tag the built Docker image with the Git reference of the workflow event. This workflow is triggered on publishing a GitHub release, so the reference will be the Git tag for the release.
