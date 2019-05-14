---
title: SAML 2.0
category_title: Overview
category: ["Integration"]
aliases: ["/ref/saml"]
---

Humio implements the SAML 2.0 _Web Browser SSO Profile_. This means authentication is delegated to an existing identity provider (IDP) which is responsible of managing user credentials. Examples of IDPs are Active Directory Federation Services (AFDS), Azure AD, Google (G Suite) and Auth0.

Leveraging an existing SSO solution in an organisation provides users of Humio with a seamless log-on experience: If they are already logged on through their SSO they will not be prompted for credentials. Instead this will be handled transparently by Humio and the IDP. This means Humio will never see the credentials of the user since the authentication is delegated to the IDP.

## Configuration

Humio needs to know the specifics about the IDP. This is configured through:

```shell
AUTHENTICATION_METHOD=saml
PUBLIC_URL=$YOUR_SERVERS_BASE_URL
SAML_IDP_SIGN_ON_URL=$IDP_SIGNON_URL        # e.g. https://accounts.google.com/o/saml2/idp?idpid=C0453
SAML_IDP_ENTITY_ID=$IDP_ENTITY_ID           # e.g. https://accounts.google.com/o/saml2?idpid=C0453
SAML_IDP_CERTIFICATE=$PATH_TO_PEM           # e.g. /home/humio/GoogleIDPCertificate-humio.com.pem
AUTO_CREATE_USER_ON_SUCCESSFUL_LOGIN=true  # default is false 
```

When a user tries to access Humio the authentication flow will start by redirecting the user to `$IDP_SIGNON_URL`. Upon a successful authentication the user will be redirected back to Humio where a Humio specific access token will be issued. For the redirects to work properly a [`PUBLIC_URL`]({{< relref "configuration/_index.md#public_url" >}}) must be configured. For details about the flow see https://en.wikipedia.org/wiki/SAML_2.0#Web_Browser_SSO_Profile.

The `$IDP_ENTITY_ID` identifies your IDP and is used internally in the authentication flow.

The provided certificate must be in PEM format (see. https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail). If you are running Humio in docker make sure the file is accessible from the container by e.g. putting it in a readonly directory as described in [docker installation]({{< ref "docker.md" >}}).

{{% notice info %}}
The redirect back to Humio is handled by the SAML Assertion Consumer Service endpoint located at `http://$HOST:$PORT/api/v1/saml/acs`. The SAML binding used in this interaction is the HTTP POST Binding. While the logon interaction from Humio to the IDP is done via a HTTP Redirect (GET) Binding.
 {{% /notice %}}
 {{% notice info %}}
 Metadata about Humio as a SAML Service Provider is available at `http://$HOST:$PORT/api/v1/saml/metadata`.
{{% /notice %}}

`AUTO_CREATE_USER_ON_SUCCESSFUL_LOGIN` if false - which is the default, users must be created in Humio before they can login. If set to true, users are auto created if they login successfully

Read more about [Configuring Humio]({{< relref "configuration/_index.md" >}})

## Access Token Lifecycle

When the SAML specific authentication flow is finished and successful a Humio access token is issued by Humio itself. Until the token expires the IDP will not be involved in authentication of the users requests. The lifetime of the access token is 24 hours.

## New User Accounts

If Humio encounters a new user that has been granted access through the IDP it will create the user in the context of Humio. For this purpose the `NameId` in the SAML authentication reponse will be used as the username property of the Humio user. The recommended username is the email.

```xml
<saml:NameID Format="urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified">Username</saml:NameID>
```

By default, the user has no rights.  So unless a user is otherwise granted access rights, he or she will not be able to do anything besides see an empty list of repos.  You can use SAML roles to control access. Otherwise, the user needs to be added explicitly as a member or admin to a repo/view to be able to access it.

## User Attribute Mapping

Instead of using `NameId` for mapping users from the IDP to Humio an user attribute name can be configured:

```shell
SAML_USER_ATTRIBUTE=email
```

In this case Humio will use the mail attribute from the IDPs SAML response after a successful authentication.

## Common Issues

Often whitelisting of Humio as an SP needs to be configured at the IDP level. Consult your vendor specific IDP documentation.
