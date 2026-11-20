# harbor-route

A GitHub Action that lets a job on a GitHub-hosted runner push to Harbor.

Harbor sits behind a Cloudflare tunnel. Its HTTP hostname caps the size of a request body
below the size of an image layer, so a push through it fails. The tunnel also carries a
TCP route, which has no such cap and admits only a Cloudflare Access service token. This
action opens that route on the runner: `cloudflared` listens on loopback, the registry's
name is pointed there, and Harbor's own TLS runs end to end. After the step, the job is
logged in and pushes to the registry by its usual name, as it would on the LAN.

## Use

```yaml
- uses: bde-dev/harbor-route@v1
  with:
    access-client-id: ${{ secrets.HARBOR_PUSH_ACCESS_CLIENT_ID }}
    access-client-secret: ${{ secrets.HARBOR_PUSH_ACCESS_CLIENT_SECRET }}
    username: ${{ vars.HARBOR_ROBOT_USERNAME }}
    password: ${{ secrets.HARBOR_ROBOT_SECRET }}

- uses: docker/build-push-action@v7
  with:
    push: true
    tags: harbor.bde-dev.net/<project>/<image>:<version>
```

Put the step before anything that builds or pushes. It sets up the buildx builder and the
registry login itself, because both depend on the route: the builder has to share the
runner's network, and it has to be created after the registry name points at loopback.

## Inputs

| Input | Required | What it is |
|---|---|---|
| `access-client-id` | yes | Access service token ID for the route |
| `access-client-secret` | yes | Access service token secret for the route |
| `username` | yes | Harbor robot account name |
| `password` | yes | Harbor robot account secret |
| `registry` | no | Registry hostname |
| `route` | no | TCP route hostname on the tunnel |

A composite action cannot read a repository's secrets, so the calling workflow passes
them in. Each repository that uses the action holds its own copies.

## Versions

`v1` is a moving tag on the newest compatible release. Renovate raises the `cloudflared`
image the action runs.
