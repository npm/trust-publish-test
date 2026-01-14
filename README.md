# @npm/trust-publish-test

Test package for validating CircleCI OIDC trusted publishing integration with npm.

## Purpose

This repository exists to test the CircleCI → npm trusted publishing flow without using long-lived secrets. It uses OIDC token exchange to obtain short-lived npm tokens for publishing.

## Setup Requirements

### 1. CircleCI Configuration

The CircleCI project must have:
- **OIDC enabled** via the CircleCI project settings
- **main-context** - CircleCI context containing any required environment variables

CircleCI Organization Details:
| Field | Value |
|-------|-------|
| Org ID | `c9035eb6-6eb2-4c85-8a81-d9ee6a1fa8c2` |
| Org Slug | `circleci/Rpg5jmswqGkxkH15Mk1Hi9` |
| Org Name | `npm-team` |
| Pipeline Definition ID | `dcd89a92-e841-46e1-afef-3ef4d3f5ea67` |

### 2. npm Trusted Publishing Configuration

⚠️ **Before publishing will work**, you must configure the trusted publisher on npm:

Go to the package settings on [npmjs.com](https://www.npmjs.com/package/@npm/trust-publish-test/access) and configure CircleCI trusted publishing with:

```
org_id: c9035eb6-6eb2-4c85-8a81-d9ee6a1fa8c2
project_id: <project-id-from-debug-workflow>
```

Use the `debug-workflow` to get the exact claim values CircleCI generates.

## Workflows

### `publish-workflow`
Publishes the package to npm. Only runs on the `main` branch.

**Flow:**
1. Get OIDC token from CircleCI with `aud: npm:registry.npmjs.org`
2. Exchange OIDC token at npm's token exchange endpoint
3. Configure npm with the received token
4. Publish the package

### `debug-workflow`
Inspects the OIDC token claims without publishing. Use this to:
- Verify CircleCI OIDC is working
- Get the exact claim values needed for npm configuration
- Debug token issues

## Token Exchange Flow

```
┌─────────────┐     ┌─────────────────┐     ┌─────────────┐
│  CircleCI   │     │  npm Registry   │     │   Package   │
│   (IdP)     │     │  (OIDC Verifier)│     │  Published  │
└─────┬───────┘     └────────┬────────┘     └──────┬──────┘
      │                      │                     │
      │  1. Request OIDC     │                     │
      │     token with aud:  │                     │
      │     npm:registry... │                     │
      │<─────────────────────│                     │
      │                      │                     │
      │  2. Return OIDC      │                     │
      │     id_token         │                     │
      │──────────────────────>                     │
      │                      │                     │
      │  3. Exchange token   │                     │
      │     POST /oidc/token │                     │
      │     /exchange/pkg... │                     │
      │──────────────────────>                     │
      │                      │                     │
      │  4. Return npm       │                     │
      │     access token     │                     │
      │<─────────────────────│                     │
      │                      │                     │
      │  5. npm publish      │                     │
      │     with token       │                     │
      │──────────────────────>                     │
      │                      │                     │
      │                      │  6. Package         │
      │                      │     published       │
      │                      │────────────────────>│
```

## Security Notes

- **No long-lived tokens** are stored in CircleCI
- Tokens are exchanged per-publish and are short-lived (typically 1 hour)
- npm validates the OIDC token against the stored trusted publisher config
- SSH reruns are rejected (`oidc.circleci.com/ssh-rerun` claim is checked)

## Related Documentation

- [CircleCI OIDC Tokens](https://circleci.com/docs/guides/permissions-authentication/openid-connect-tokens/)
- [npm Trusted Publishing](https://docs.npmjs.com/trusted-publishers)
