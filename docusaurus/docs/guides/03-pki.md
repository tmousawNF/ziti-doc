---
id: pki
title: Public Key Infrastructure (PKI)
sidebar_label: PKI
---

import PkiTroubleshootingMd from './05-troubleshooting/_pki-troubleshooting.md'

All Ziti Networks leverage [Public Key Infrastructure (PKI)](https://en.wikipedia.org/wiki/Public_key_infrastructure) to
provide secure network connections.  This page is not intended to be a comprehensive guide. What it is, is a set of
rules that must be followed to properly configure a Ziti Network. If there are issues when connecting any portion
of a Ziti Network to another - this page should serve as a starting point of understanding.

:::note
PKI is a complex topic and it is recommended to be familiar with what is a PKI as well as how to properly use and
configure a one before making any decisions about the PKI the Ziti Network uses.
:::

The Ziti Network allows the operator to declare any trust anchors as valid. This means Ziti does not need to be
configured with a full chain of certificates which link fully back to a root CA. A configuration using a full chain back
to a root CA is of course supported but it is not explicitly required.  This allows the operator to configure a Ziti
Network using one or more chains of trust back to the provided trust anchors.  The sections below will describe where
these trust anchors can be configured.

Ziti Network components are required to present a certificate to other Ziti Network components during the connection
establishment. This certificate will need to be valid per the configured trust anchor store being connected to.

### Ziti Controller

The Ziti Controller has three distinct sections related to PKI: `identity`, `edge.api.identity`,
`edge.enrollment.signingCert`. The `edge.api.identity` configuration section is optional and is provided to allow the external
REST endpoint to present a certificate that is different than the one configured in the identity section.

Connections to the Ziti Controller are considered valid if the certificate presented during connection is signed by a
trust anchor declared within the identity.ca configuration or if the certificate presented is signed by the certificate
specified in the `edge.enrollment.signingCert`.

#### PKI Configuration

The identity section of the Ziti Controller configuration is used by the Ziti Controller when connections are
established to or from other components of a Ziti Network. There are four sections in the identity block:
`cert`, `server_cert`, `key`, `ca`.

**ca**: A file representing a group of certificates with one or more certificate chains terminating at a trust anchor.
When a Ziti Network component connects to the Ziti Controller and offers a certificate for validation the incoming
connection is checked to see if it signed by a trust anchor specified in this file.

**key**: Also referred to as the [private key](https://en.wikipedia.org/wiki/Symmetric-key_algorithm). It is generated
first and used to produce the certificates specified in the `cert` and `server_cert` fields of the Ziti Controller
configuration file.

**cert**: The certificate presented to other Ziti Network components during connection establishment.

**server_cert**: The certificate returned by the Ziti Controller when other Ziti Network components attempt to
communicate to the Ziti Controller over the IP and port specified in the `ctrl.listener` or `mgmt.listener` fields of the Ziti Controller
configuration file. If an edge section is present in the configuration file and no edge.api.identity section exists this
certificate is also returned to incoming connections to the `edge.api.advertise` endpoint.

### Edge Router

An Edge Router has one section related to PKI: `identity`. It is important to note that an Edge Router will
manage its own PKI. Allowing the Edge Router to manage its own PKI is almost certainly desired. The
only setting that an operator may wish to provide is the `key` field of the identity. This field is treated differently
than the other values specified.  If the `key` specified does not exist a new key will be generated. If the `key`
provided exists the Edge Router will use it and the other fields will be **regenerated and overwritten** as necessary.

The certificate generated will be signed by the Ziti Controller using the certificate specified in `edge.enrollment.signingCert`.

#### PKI Configuration

The `identity` section of the Edge Router configuration is used by the Edge Router when connections are
established to or from the other components of a Ziti Network. There are four sections in the identity block:
`cert`, `server_cert`, `key`, `ca`.

**ca**: A file representing a group of certificates with one or more certificate chains terminating at a trust anchor.
When a Ziti Network component connects to the Edge Router and offers a certificate for validation the incoming
connection is checked to see if it signed by a trust anchor specified in this file.

**key**: Also referred to as the [private key](https://en.wikipedia.org/wiki/Symmetric-key_algorithm). It is generated
first and used to produce the certificates specified in the `cert` and `server_cert` fields of the Edge Router
configuration file.

**cert**: The certificate presented to other Ziti Network components during connection establishment.

**server_cert**: The certificate returned by the Edge Router when other Ziti Network components attempt to
communicate to the Edge Router over the IP and port specified in the `ctrl.listener` or `mgmt.listener` fields of the Edge Router
configuration file.

### Third Party CA (optional)

A third party CA is one which is maintained and managed entirely outside of the Ziti Network. This is an important
feature for organizations wishing to administer and maintain the certificates used by the different pieces of the Ziti
Network. A Ziti Network is capable of using third party PKIs as the trust mechanism for enrollment and authentication of
clients for a Ziti Network.

With the PKI being managed externally a Ziti Network is never in possession of the private key. This means the Ziti
Network cannot maintain nor distribute certificates necessary for creating secure connections. The Ziti Network is
only capable of verifying if the certificate presented was signed by the externally managed PKI.

Maintaining a PKI outside of the Ziti Network is a more complex configuration. If a PKI is already established
and maintained externally setting up a Ziti Network with a third party CA may be desired.

#### Registering the CA

A Ziti Network will not trust any third party CA implicitly. Before a third party CA can be used for enrollment and
authentication of clients in a Ziti Network it must be registered with the Ziti Controller to ensure certificates signed
by the third party CA can be trusted.  

Registering a third party CA is done by using the REST endpoint `/cas` from the Ziti Controller. To register a third
party CA the following information is required to be posted to the endpoint:

* **name**: the desired name of the CA
* **isEnrollmentEnabled**: a boolean value indicating if the CA can be used for enrollment. Defaults to true. Set to false
  to prevent further enrollments using this CA
* **isAuthEnabled**: a boolean value indicating if the CA can be used for authentication. Defaults to true. Set to false to
  prevent all authentication from endpoints signed by this certificate

Assuming the create request was well formed and successful, the response from this invocation will contain a field
representing the `id` of the third party CA at `data.id`. The id of the third party CA will be needed when validating
the third party CA.

#### Validating the CA

After being submitted to the Ziti Controller, the third party CA will have the isCsrValidated field set to false
indicating it is not yet ready for use. A second step is needed to ensure the third party CA is setup properly as a CA.
This step ensures the third party CA provided is capable of fulfilling CSR requests. Clients attempting to connect to a
Ziti Network using the third party CA will be rejected.

To validate the third party CA a CSR must be generated and fulfilled by the third party CA to generate a certificate
with the common name (CN) field set to a value assigned by the Ziti Controller. The Ziti Controller `/cas`
REST endpoint can be interrogated to retrieve the details for a specific third party CA. The field necessary to validate
the third party CA is `data.verificationToken` and is obtained at this endpoint. A certificate is then created and
signed by the third party CA with the common name field set to the verificationToken.

To finish verifying the third party CA, the certificate created with the verificationToken is posted back to the Ziti
Controller at `/cas/${id}/verify`. The `id` is also obtained during the creation process. After posting the certificate
with the `verificationToken` as the common name the third party CA will change from `isVerified=false` to `isVerified=true`.

<PkiTroubleshootingMd />