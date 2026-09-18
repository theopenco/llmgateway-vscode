# Releasing

Publishing is automated: pushing a `v*` tag builds the VSIX and publishes it to the
VS Marketplace and Open VSX, then attaches it to a GitHub release.

## One-time setup

1. **VS Marketplace publisher** — sign in to <https://marketplace.visualstudio.com/manage>
   with the Microsoft/Azure DevOps account that should own the listing and create the
   publisher `llmgateway` (must match `publisher` in `package.json`).
2. **`VSCE_PAT`** — in the same Azure DevOps organization, create a personal access token
   (<https://dev.azure.com> → User settings → Personal access tokens) with
   **Organization: All accessible organizations** and scope **Marketplace → Manage**.
   Store it as the repository secret `VSCE_PAT`.
3. **Open VSX (optional, powers Cursor/Windsurf/VSCodium)** — sign in at
   <https://open-vsx.org> with GitHub, agree to the publisher agreement, create an access
   token, then claim the namespace once:

   ```sh
   pnpm exec ovsx create-namespace llmgateway -p <token>
   ```

   Store the token as the repository secret `OVSX_PAT`. Without it the Open VSX step is
   skipped.

## Cutting a release

```sh
# bump "version" in package.json and add a CHANGELOG.md section
git commit -am "chore: release v0.2.0"
git tag v0.2.0
git push origin main --tags
```

The tag must match `package.json`'s version or the workflow fails before publishing.

## Publishing by hand

```sh
pnpm package                                     # llmgateway-vscode-<version>.vsix
pnpm exec vsce publish --no-dependencies --packagePath llmgateway-vscode-<version>.vsix
pnpm exec ovsx publish llmgateway-vscode-<version>.vsix --pat <token>
```
