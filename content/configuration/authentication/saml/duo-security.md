---
title: Duo security
menuTitle: Duo Security
category: ["Integration"]
aliases: ["/ref/duo"]
---

[Duo security](https://duo.com) provides a great way of authenticating your users for your on-prem Humio installation.
With the addition of SAML in version 1.1.27 Humio now also supports Duo.

## Prerequisites

Before configuring SAML authentication a few things needs to be in place:

* [Duo Access Gateway (DAG)](https://duo.com/docs/dag) installed and configured with at least one Authentication Source
* Familiarize yourself with Humio's [configuration properties]({{< ref "configuration" >}})
* Make sure you have one root accounts added. Typically by adding your email address in the [administration section]({{< ref "/configuration/user-management.md" >}}).

## Steps:

1. Open your DAG and take note of the following parameters from Applications page
  * "SSO URL"
  * "Entity ID"
  * Save the certificate to a known location on your Humio host
2. Change the following configuration properties in Humio
  * `AUTHENTICATION_METHOD=saml`
  * `PUBLIC_URL` [^1]
  * `SAML_IDP_SIGN_ON_URL` to the value of "SSO URL" from the DAG
  * `SAML_IDP_ENTITY_ID` to the value of "Entity ID" from the DAG
  * `SAML_IDP_CERTIFICATE` with the location of your DAG certificate [^2]
3. Restart Humio
4. Read the output of `http://$HOST:$PORT/api/v1/saml/metadata` and take notes of the following values
  * `md:EntityDescriptor#entityID`, which should be a url starting with your `PUBLIC_URL` followed by `/api/v1/saml/metadata`
  * `md:AssertionConsumerService#Location`, which should be a url starting with your `PUBLIC_URL` followed by `/api/v1/saml/acs`
5. Log into your Duo account and add a new ["Generic SAML Service Provider"](https://duo.com/docs/dag-generic), where
  * "Entity ID" is the value of `md:EntityDescriptor#entityID`
  * "Assertion Consumer Service" is the value of `md:AssertionConsumerService#Location`
  * "NameID attribute" should be set to `email`
6. [Save the configuration file](https://duo.com/docs/dag-generic#create-your-cloud-application-in-duo) and [upload it to your DAG](https://duo.com/docs/dag-generic#add-your-cloud-application-to-duo-access-gateway)


[^1]: See explanation in the [configuration properties]({{< ref "configuration" >}})
[^2]: If running the Docker image, please make sure you have mounted a certs volume by adding the following volume `-v /certs:/certs:ro`
