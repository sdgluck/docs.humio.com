---
title: Azure AD using SAML
menuTitle: Azure AD
category: ["Integration"]
aliases: ["/ref/azure-ad"]
---

### Login with Azure AD using SAML and prefixes based on roles

The following example works with Azure AD.

This set of settings allow plain login using SAML, without managing membership of repositories from SAML, but leaving those within Humio, as is the default.

```
# You'll get this from the SAML screen in AAD
SAML_IDP_SIGN_ON_URL=https://login.microsoftonline.com/<YOUR_AAD_GUID>/saml2

# You'll get this from the SAML screen in AAD
SAML_IDP_ENTITY_ID=https://sts.windows.net/<YOUR_AAD_GUID>/

# Once you upload your cert to your Enterprise App in AAD, you'll download this response file 
SAML_IDP_CERTIFICATE=/certs/humio-AAD-SSO.pem
```

#### Mapping SAML roles to prefix queries (If using role-based authorization)

To also apply prefixes and repo memberships based on roles in AAD, these additional settings are required. See [role-based authoriation]({{< relref "configuration/authentication/role-based-auth.md" >}}) for details on how mapping roles to prefixes work and the format of the configration files.
```
# You'll need to create this attribute in your AAD Enterprise App and map it to user.assignedroles
SAML_GROUP_MEMBERSHIP_ATTRIBUTE=http://schemas.microsoft.com/ws/2008/06/identity/claims/role

AUTO_UPDATE_GROUP_MEMBERSHIPS_ON_SUCCESSFUL_LOGIN=true
PREFIX_AUTHORIZATION_ENABLED=true
```

### Creating the app in Azure Active Directory
1. Login to https://portal.azure.com
* Open Enterprise Applications
* Create a new app
* Single Sign-on > select SAML
* Edit Basic SAML Configuration
1. Your Identifier / Entity ID will be `<YOUR_SITE_URL>/api/v1/saml/metadata`
2. Your Reply URL will be `<YOUR_SITE_URL>/api/v1/saml/acs`
3. Leave Sign-on url and Relay state blank
* Edit User Attributes and Claims
1. Add a new claim
2. Name = `role`
3. Source attribute = `user.assignedroles`
4. Leave Namespace blank
* Edit SAML signing certificate
1. Import certificate
2. Upload your pfx certificate > it needs to be encrypted with a password to add
3. CLick the 3 dots next to your new cert and set it to active
4. Click the three dots next to your new cert and download a PEM cert
5. You can delete the old cert here if you want
6. Download the pem cert to your humio server and place it where you place the rest of your certs
7. Edit your humio.conf file to include the path to this cert in `SAML_IDP_CERTIFICATE`
* Set up Humio by copying values to `humio.conf`
1. Copy the value from the Login URL box to `SAML_IDP_SIGN_ON_URL`
2. Copy the value from the Azure AD Identiier to `SAML_IDP_ENTITY_ID`

### Create Roles (If using role-based authorization)

1. In AAD, go to App Registrations
* Open the app you just created
* Edit the manifest
* Under ` "appRoles": [` add the following, at the bottom, just above the closing `]`

```
    {
      "allowedMemberTypes": [
        "User"
      ],
      "displayName": "HumioAdministrator",
      "id": "30f71b1a-74db-4a0f-bae6-dcdd4bc8a57d",
      "isEnabled": true,
      "description": "Administrators can access all Repos",
      "value": "HumioAdministrator"
    },
    {
      "allowedMemberTypes": [
        "User"
      ],
      "displayName": "HumioUser",
      "id": "4722356f-1b76-4a8c-8c1e-91282e21affe",
      "isEnabled": true,
      "description": "Users can access all Repos",
      "value": "HumioUser"
    },
```

Ensure your indentations are good, then save.

#### Assign Users

1. Open your app under Enterprise Applications
* Select Users and Groups
* Assign users (or groups) to your app 
* If using roles, assign the user (or group) to one of the roles you just created

#### Create authorizations for your roles (If using role-based authorization)

Save the sample 'view-role-prefix-auth.json' file from the top of this page to your humio data directory (Usually `/data/humio-data`)

### Rebuild your instance

If using docker, make sure to recreate the Humio instance at this point in order to get the new configuration files included.

### Test

Give it a test drive! Users should be automatically created and assigned the roles specified in AAD.

### Contributors
Thanks to [Paul](https://github.com/pyoungberg) for contributing to this page.
