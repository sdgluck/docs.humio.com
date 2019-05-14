---
title: Authenticating with OAuth 2.0
menuTitle: OAuth 2.0
category: ["Integration"]
pageImage: /integrations/oauth.svg
aliases: ["/ref/oauth"]
---

Humio supports the OAuth 2.0 login flow for the following providers:

- [Google Sign-In]({{< ref "#google" >}})
- [GitHub Sign-In]({{< ref "#github" >}})
- [BitBucket Sign-In]({{< ref "#bitbucket" >}})

Providers must be configured on the Humio server, as seen in the section
for each provider.

You can enable several providers at the same time by setting multiple provider
configurations.

Before you get started you must create OAuth Apps with the provider and
get `client_id` and `client_secret`, and configure your `redirect_uri`.

{{% notice warning %}}
In order for OAuth authentication to work properly you must provide
a URL where Humio can be reached from the browser, see the configuration
option [`PUBLIC_URL`]({{< relref "configuration/_index.md#public_url" >}}).
{{% /notice %}}

## Google Sign-In {#google}

Detailed Setup Instructions: https://developers.google.com/identity/sign-in/web/sign-in

### Quick Summary

- Create a Project from the Google Developer Console,
- Create a _OAuth Client ID_ on the Credentials Page,
- Add an _Authorized redirect URI_: `%PUBLIC_URL%/auth/google`

Where [`%PUBLIC_URL%`]({{< relref "configuration/_index.md#public_url" >}}) is the same value as Humio is configured with.
This can e.g. be `http://localhost:8080/auth/google` during development.
Login will fail if the `redirect_uri` is not set correctly.

Once your app is created you can configure Humio to use authenticate with Google:

### Configuration Properties

```shell
AUTHENTICATION_METHOD=oauth
PUBLIC_URL=$YOUR_SERVERS_BASE_URL
GOOGLE_OAUTH_CLIENT_ID=$CLIENT_ID #The client_id from your Google OAuth App
GOOGLE_OAUTH_CLIENT_SECRET=$CLIENT_SECRET The #client_secret your Google OAuth App
AUTO_CREATE_USER_ON_SUCCESSFUL_LOGIN=true  # default is false
```

Read more about [Configuring Humio]({{< relref "configuration/_index.md" >}})

## GitHub Sign-In {#github}

Setup Instructions: https://developer.github.com/apps/building-oauth-apps/creating-an-oauth-app/

### Quick Summary

- Create an OAuth App from your organization / user settings page,
- Set the _Authorization callback URL_: `%PUBLIC_URL%/auth/github`

Read more about [Configuring Humio]({{< relref "configuration/_index.md" >}})

Once your app is created you can configure Humio to use authenticate with GitHub:

### Configuration Properties

```shell
AUTHENTICATION_METHOD=oauth
PUBLIC_URL=$YOUR_SERVERS_BASE_URL
GITHUB_OAUTH_CLIENT_ID=$CLIENT_ID # The client_id from your GitHub OAuth App
GITHUB_OAUTH_CLIENT_SECRET=$CLIENT_SECRET # The client_secret your GitHub OAuth App
AUTO_CREATE_USER_ON_SUCCESSFUL_LOGIN=true  # default is false
```

Read more about [Configuring Humio]({{< relref "configuration/_index.md" >}})

## BitBucket Sign-In {#bitbucket}

Setup Instructions: https://confluence.atlassian.com/bitbucket/integrate-another-application-through-oauth-372605388.html

### Quick Summary

- Go to your Account Settings
- Create an OAuth Consumer
- Set the _Callback URL_: `%PUBLIC_URL%/auth/bitbucket`
- Grant the `account:email` permission.
- Save
- Find the Key (Client Id), and Secret (Client Secret) in the list of consumers.

Read more about [Configuring Humio]({{< relref "configuration/_index.md" >}})

Once your consumer is created you can configure Humio to use authenticate with BitBucket:

### Configuration Properties

```shell
AUTHENTICATION_METHOD=oauth
PUBLIC_URL=$YOUR_SERVERS_BASE_URL
BITBUCKET_OAUTH_CLIENT_ID=$CLIENT_ID # The Key from your BitBucket OAuth Consumer
BITBUCKET_OAUTH_CLIENT_SECRET=$CLIENT_SECRET # The Secret your BitBucket OAuth Consumer
AUTO_CREATE_USER_ON_SUCCESSFUL_LOGIN=true  # default is false
```

Read more about [Configuring Humio]({{< relref "configuration/_index.md" >}})
