# Releasing

Publishing is automated: pushing a `v*` tag builds the VSIX and publishes it to the
VS Marketplace and Open VSX, then attaches it to a GitHub release.

## Cutting a release

```sh
# bump "version" in package.json and add a CHANGELOG.md section
git commit -am "chore: release v0.2.0"
git tag v0.2.0
git push origin main --tags
```

The tag must match `package.json`'s version or the workflow fails before publishing.

## Credentials

Both are repository secrets. The publisher id `llmgateway` must keep matching
`publisher` in `package.json`.

| Secret     | Registry       | Required                            |
| ---------- | -------------- | ----------------------------------- |
| `VSCE_PAT` | VS Marketplace | yes                                 |
| `OVSX_PAT` | Open VSX       | no — the step is skipped when unset |

### Rotating `VSCE_PAT`

An Azure DevOps personal access token from the account that owns the publisher
(<https://dev.azure.com> → User settings → Personal access tokens), with
**Organization: All accessible organizations** and scope **Marketplace → Manage**.
Verify before storing it:

```sh
VSCE_PAT=<token> pnpm exec vsce verify-pat llmgateway
gh secret set VSCE_PAT --repo theopenco/llmgateway-vscode
```

The current token expires 2027-09-17, but note that Azure DevOps
[stops supporting PATs scoped to all accessible organizations on 2026-12-01](https://aka.ms/GlobalPATDeprecation),
so this will need revisiting before then.

### Rotating `OVSX_PAT`

Sign in at <https://open-vsx.org> with GitHub, agree to the publisher agreement, and
create an access token. The `llmgateway` namespace is claimed once:

```sh
pnpm exec ovsx create-namespace llmgateway -p <token>
```

## Publishing by hand

```sh
pnpm package                                     # llmgateway-vscode-<version>.vsix
pnpm exec vsce publish --no-dependencies --packagePath llmgateway-vscode-<version>.vsix
pnpm exec ovsx publish llmgateway-vscode-<version>.vsix --pat <token>
```

## Marketplace listing

The publisher profile (name, logo, links) is managed at
<https://marketplace.visualstudio.com/manage/publishers/llmgateway>, separately from the
extension listing, which comes from `package.json` and `README.md` in the VSIX.
Verifying ownership of `llmgateway.io` there adds the verified badge next to the
publisher name and requires a DNS TXT record.
