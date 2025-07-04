---
title: "BookStack"
description: "Integrating BookStack with the Authelia OpenID Connect 1.0 Provider."
summary: ""
date: 2022-06-15T17:51:47+10:00
draft: false
images: []
weight: 620
toc: true
support:
  level: community
  versions: true
  integration: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  noindex: false # false (default) or true
---

## Tested Versions

- [Authelia]
  - [v4.38.0](https://github.com/authelia/authelia/releases/tag/v4.38.0)
- [BookStack]
  - [v23.02.2](https://github.com/BookStackApp/BookStack/releases/tag/v23.02.2)

{{% oidc-common %}}

### Assumptions

This example makes the following assumptions:

- __Application Root URL:__ `https://bookstack.{{< sitevar name="domain" nojs="example.com" >}}/`
- __Authelia Root URL:__ `https://{{< sitevar name="subdomain-authelia" nojs="auth" >}}.{{< sitevar name="domain" nojs="example.com" >}}/`
- __Client ID:__ `bookstack`
- __Client Secret:__ `insecure_secret`

Some of the values presented in this guide can automatically be replaced with documentation variables.

{{< sitevar-preferences >}}

{{< callout context="caution" title="Important Note" icon="outline/alert-triangle" >}}
[BookStack](https://www.bookstackapp.com/) does not properly URL encode the secret per [RFC6749 Appendix B](https://datatracker.ietf.org/doc/html/rfc6749#appendix-B) at the time this
article was last modified (noted at the bottom). This means you'll either have to use only alphanumeric characters for
the secret or URL encode the secret yourself.
{{< /callout >}}

## Configuration

### Authelia

The following YAML configuration is an example __Authelia__ [client configuration] for use with [BookStack] which will
operate with the application example:

```yaml {title="configuration.yml"}
identity_providers:
  oidc:
    ## The other portions of the mandatory OpenID Connect 1.0 configuration go here.
    ## See: https://www.authelia.com/c/oidc
    clients:
      - client_id: 'bookstack'
        client_name: 'BookStack'
        client_secret: '$pbkdf2-sha512$310000$c8p78n7pUMln0jzvd4aK4Q$JNRBzwAo0ek5qKn50cFzzvE9RXV88h1wJn5KGiHrD0YKtZaR/nCb2CJPOsKaPK0hjf.9yHxzQGZziziccp6Yng'  # The digest of 'insecure_secret'.
        public: false
        authorization_policy: 'two_factor'
        require_pkce: false
        pkce_challenge_method: ''
        redirect_uris:
          - 'https://bookstack.{{< sitevar name="domain" nojs="example.com" >}}/oidc/callback'
        scopes:
          - 'openid'
          - 'profile'
          - 'email'
        response_types:
          - 'code'
        grant_types:
          - 'authorization_code'
        access_token_signed_response_alg: 'none'
        userinfo_signed_response_alg: 'none'
        token_endpoint_auth_method: 'client_secret_basic'
```

### Application

To configure [BookStack] there is one method, using the [Environment Variables](#environment-variables).

#### Environment Variables

To configure [BookStack] to utilize Authelia as an [OpenID Connect 1.0] Provider, use the following environment
variables:

##### Standard

```shell {title=".env"}
AUTH_METHOD=oidc
OIDC_ISSUER=https://{{< sitevar name="subdomain-authelia" nojs="auth" >}}.{{< sitevar name="domain" nojs="example.com" >}}
OIDC_ISSUER_DISCOVER=true
OIDC_CLIENT_ID=bookstack
OIDC_CLIENT_SECRET=insecure_secret
OIDC_NAME=Authelia
OIDC_DISPLAY_NAME_CLAIMS=name
```

##### Docker Compose

```yaml {title="compose.yml"}
services:
  bookstack:
    environment:
      AUTH_METHOD: 'oidc'
      OIDC_ISSUER: 'https://{{< sitevar name="subdomain-authelia" nojs="auth" >}}.{{< sitevar name="domain" nojs="example.com" >}}'
      OIDC_ISSUER_DISCOVER: 'true'
      OIDC_CLIENT_ID: 'bookstack'
      OIDC_CLIENT_SECRET: 'insecure_secret'
      OIDC_NAME: 'Authelia'
      OIDC_DISPLAY_NAME_CLAIMS: 'name'
```

## See Also

- [BookStack OpenID Connect Documentation](https://www.bookstackapp.com/docs/admin/oidc-auth/)

[Authelia]: https://www.authelia.com
[BookStack]: https://www.bookstackapp.com/
[OpenID Connect 1.0]: ../../openid-connect/introduction.md
[client configuration]: ../../../configuration/identity-providers/openid-connect/clients.md
