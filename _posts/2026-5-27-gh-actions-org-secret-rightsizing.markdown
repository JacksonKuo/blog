---
layout: post
title: "GitHub Defense: Part II - Org Secret Rightsizing"
date: 2026-5-27
tags: ["github"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement
Document the process for migrating GitHub Actions Org Secrets from `all` or `private/internal` access to `selected` repositories.

# REST APIs and GH CLI
* [create-or-update-an-organization-secret](https://docs.github.com/en/rest/actions/secrets?apiVersion=2026-03-10#create-or-update-an-organization-secret)
    * *"Secrets" organization permissions (write)*
    * *encrypted_value string **Required***
```
# not real secret values, from GitHub's own documentation
gh api \
    --method PUT \
    -H "Accept: application/vnd.github+json" \
    -H "X-GitHub-Api-Version: 2026-03-10" \
    /orgs/ORG/actions/secrets/SECRET_NAME \
    --input - <<< '{
        "encrypted_value": "c2VjcmV0",
        "key_id": 12345678912345678,
        "visibility": "selected",
        "selected_repository_ids": [
            1296269,
            1296280
        ]
    }'
```
* [set-selected-repositories-for-an-organization-secret](https://docs.github.com/en/rest/actions/secrets?apiVersion=2026-03-10#set-selected-repositories-for-an-organization-secret)
    * ***Replaces all repositories for an organization secret***
    * *"Secrets" organization permissions (write)*
```
gh api \
    --method PUT \
    -H "Accept: application/vnd.github+json" \
    -H "X-GitHub-Api-Version: 2026-03-10" \
    /orgs/ORG/actions/secrets/SECRET_NAME/repositories \
    --input - <<< '{
        "selected_repository_ids": [
            64780797
        ]
    }'
```
* [add-selected-repository-to-an-organization-secret](https://docs.github.com/en/rest/actions/secrets?apiVersion=2026-03-10#add-selected-repository-to-an-organization-secret)
    * *"Secrets" organization permissions (write) and "Metadata" repository permissions (read)*
```
gh api \
    --method PUT \
    -H "Accept: application/vnd.github+json" \
    -H "X-GitHub-Api-Version: 2026-03-10" \
    /orgs/ORG/actions/secrets/SECRET_NAME/repositories/REPOSITORY_ID
```
* [get-an-organization-public-key](https://docs.github.com/en/rest/actions/secrets?apiVersion=2026-03-10#get-an-organization-public-key)
    * *"Secrets" organization permissions (read)*
```
gh api \
  -H "Accept: application/vnd.github+json" \
  -H "X-GitHub-Api-Version: 2026-03-10" \
  /orgs/ORG/actions/secrets/public-key
```
* [example-encrypting-a-secret-using-python](https://docs.github.com/en/rest/guides/encrypting-secrets-for-the-rest-api?apiVersion=2026-03-10#example-encrypting-a-secret-using-python)
    * [PyNaCl](https://pynacl.readthedocs.io/en/latest/public/#nacl-public-sealedbox)

```python
from base64 import b64encode
from nacl import encoding, public

def encrypt(public_key: str, secret_value: str) -> str:
    """Encrypt a Unicode string using the public key."""
    public_key = public.PublicKey(public_key.encode("utf-8"), encoding.Base64Encoder())
    sealed_box = public.SealedBox(public_key)
    encrypted = sealed_box.encrypt(secret_value.encode("utf-8"))
    return b64encode(encrypted).decode("utf-8")
encrypt("YOUR_BASE64_KEY", "YOUR_SECRET")
```
* [https://cli.github.com/manual/gh_secret_set](https://cli.github.com/manual/gh_secret_set)
    * `gh secret set MYSECRET --org myOrg --visibility all`
    * `gh secret set MYSECRET --org myOrg --repos repo1,repo2,repo3`
    * `gh secret set MYSECRET --org myOrg --no-repos-selected`

Hmm, i think that's it. 

# Setup
I assume you can switch the visiblity without passing in the credentials again, since that's possible in the UI. 

Well, let's test it out. 

1. Create a PAT with org secret write
2. Create a new org secret: [https://github.com/organizations/jkuo-org/settings/secrets/actions](https://github.com/organizations/jkuo-org/settings/secrets/actions)

Hmm, the REST API and GH CLI documentation looks like you have to re-pass in the secret. Wtf. This is noted in a few places as a feature request:
* [#6327](https://github.com/cli/cli/issues/6327#issuecomment-1500527688)
* [#9383](https://github.com/cli/cli/issues/9383#issue-2434956309)
* [#9808](https://github.com/cli/cli/issues/9808)

# Learnings
Well that's pretty bad. The UI endpoint [https://github.com/organizations/jkuo-org/settings/secrets/actions/](https://github.com/organizations/jkuo-org/settings/secrets/actions/) allows changing the visibility without knowing the secret. This is the underlying UI request, with the `encrypted_value` not set:

```
POST /organizations/jkuo-org/settings/secrets/actions/ORG_SECRET_TEST
Host: github.com

method=put&authenticity_token=tktk&encrypted_value=&
key_id=3380204578043523366&visibility=VISIBILITY_SELECTED_REPOSITORIES
```

And here's the difference when including 1 or more repos:

One repository
```xml
_method=put&
authenticity_token=&
encrypted_value=&
key_id=3380204578043523366&
visibility=VISIBILITY_SELECTED_REPOSITORIES&
repository_ids%5B%5D=R_kgDOQH9dZQ`    
```

Multiple repositories
```xml
_method=put&
authenticity_token=&
encrypted_value=&
key_id=3380204578043523366&
visibility=VISIBILITY_SELECTED_REPOSITORIES&
repository_ids%5B%5D=R_kgDOQH9dZQ&
repository_ids%5B%5D=R_kgDOO4Zqhw
```

However there's no corresponding functionality in the REST API. And there's no secrets functionality in the GraphQL API[^1].

Attempts to use a blank `""` or an empty `encrypted_value` return errors:
```json
curl -L \
  -X PUT \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer $GITHUB_TOKEN" \
  -H "X-GitHub-Api-Version: 2026-03-10" \
  https://api.github.com/orgs/jkuo-org/actions/secrets/ORG_SECRET_TEST \
  -d '{"encrypted_value":,"key_id":"3380204578043523366","visibility":"selected","selected_repository_ids":[1082088805]}'

{
  "message": "Problems parsing JSON",
  "documentation_url": "https://docs.github.com/rest/actions/secrets#create-or-update-an-organization-secret",
  "status": "400"
}
```

```json
curl -L \
  -X PUT \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer $GITHUB_TOKEN" \
  -H "X-GitHub-Api-Version: 2026-03-10" \
  https://api.github.com/orgs/jkuo-org/actions/secrets/ORG_SECRET_TEST \
  -d '{"encrypted_value":"","key_id":"3380204578043523366","visibility":"selected","selected_repository_ids":[1082088805]}'

{
  "message": "Invalid request.\n\n does not match /^(?:[A-Za-z0-9+\\/]{4})*(?:[A-Za-z0-9+\\/]{2}==|[A-Za-z0-9+\\/]{3}=|[A-Za-z0-9+\\/]{4})$/.",
  "documentation_url": "https://docs.github.com/rest/actions/secrets#create-or-update-an-organization-secret",
}
```
And attempts to call the `set-selected-repositories-for-an-organization-secret` return the error: 

```json
curl -L \
  -X PUT \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer $GITHUB_TOKEN" \
  -H "X-GitHub-Api-Version: 2026-03-10" \
  https://api.github.com/orgs/jkuo-org/actions/secrets/ORG_SECRET_TEST/repositories \
  -d '{"selected_repository_ids":[1082088805]}'

{
  "message": "Conflict",
  "errors": "You cannot update selected repositories for a secret when the visibility is not set to 'selected'",
  "documentation_url": "https://docs.github.com/rest/actions/secrets#set-selected-repositories-for-an-organization-secret",
  "status": "409"
}
```

Weirdly, it looks like right now the only way to change the visiblity is manually or via an undocumented call using a bummed cookie + `authenticity_token`. 

There's a bunch of changes coming via the GitHub Actions 2026 Security Roadmap: [https://github.blog/news-insights/product-news/whats-coming-to-our-github-actions-2026-security-roadmap/](https://github.blog/news-insights/product-news/whats-coming-to-our-github-actions-2026-security-roadmap/). However, i don't think there's anything that fixes this directly. And terraform or safe-settings[^2] doesn't really help this situation.

# Solution
I guess the step-by-step to change the visibility from `all` or `private/internal` to `selected` is:

* Limited Repos List
    * change visibility and selected repos via clickops
* Large Repos List
    * change visibility via clickops
    * update selected repos
        * clickops
        * call `set-selected-repositories-for-an-organization-secret` once, as this call deletes the existing repo list
* ~~Session Riding~~
    * get cookie
    * handle CSRF
    * maybe automate using playwright, ugh
    * update selected repos
        * call `add-selected-repository-to-an-organization-secret` multiple times
        * call `set-selected-repositories-for-an-organization-secret` once, as this call deletes the existing repo list

# Add Repos
So changing the visiblity is a no-go, but what about just adding or removing a single repo for a org secret that is already set to `selected`. Let's get the repo ID for `repo-test`

`gh api repos/jkuo-org/repo-test` > 1082088805

```
curl -L \
  -X PUT \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer $GITHUB_TOKEN" \
  -H "X-GitHub-Api-Version: 2026-03-10" \
  https://api.github.com/orgs/jkuo-org/actions/secrets/ORG_SECRET_TEST/repositories/1082088805
```

That was successful. 

# References
[^1]: [https://docs.github.com/en/graphql/reference/mutations](https://docs.github.com/en/graphql/reference/mutations)

[^2]: [https://github.com/github/safe-settings](https://github.com/github/safe-settings)