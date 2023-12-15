This is a minimal reproduction as requested here:
https://github.com/renovatebot/renovate/discussions/26253

We use corepack to keep the version of our used package manager yarn the same
over all environments. Since our network traffic from within our pipelines is
restricted, the current yarn bin is included in this repository and referenced
in `.yarnrc.yml` via `yarnPath`. This may not be the recommended method anymore
but still works fine. For us, there is currently no other way as it is not
possible (yet) to provide corepack with an alternative download source for
yarn.

We use the renovate docker image `renovate:latest`.

# current behavior

When renovate creates an update branch, it fails on updating the `yarn.lock`
file with the message:

```
 WARN: artifactErrors (repository=renovate-corepack-yarnpath-reproduction, branch=renovate-major-cypress)
       "artifactErrors": [
         {
           "lockFile": "yarn.lock",
           "stderr": "Internal Error: Server answered with HTTP 403 when performing the request to https://repo.yarnpkg.com/4.0.2/packages/yarnpkg-cli/bin/yarn.js; for troubleshooting help, see https://github.com/nodejs/corepack#troubleshooting\n    at ClientRequest.<anonymous> (/opt/containerbase/tools/corepack/0.23.0/node_modules/corepack/dist/lib/corepack.cjs:42192:21)\n    at Object.onceWrapper (node:events:629:26)\n    at ClientRequest.emit (node:events:514:28)\n    at HTTPParser.parserOnIncomingClient (node:_http_client:693:27)\n    at HTTPParser.parserOnHeadersComplete (node:_http_common:119:17)\n    at Socket.socketOnData (node:_http_client:535:22)\n    at Socket.emit (node:events:514:28)\n    at Readable.read (node:internal/streams/readable:758:10)\n    at Socket.read (node:net:771:39)\n    at flow (node:internal/streams/readable:1248:53)\n"
         }
       ]
```

Even though the yarn bin is provided inside the repository, corepack is not
able to find it during a renovate run and therefore tries to download it,
which failes due to restricted network.

In an unrestricted network, the download should fail if
`COREPACK_ENABLE_NETWORK` is set to 0.

# expected behavior

- Corepack does not try to download yarn.
- `yarn.lock` is updated using the provided yarn bin.
