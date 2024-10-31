# Chapter 25. Secure Configuration
>helps keep and use sensitive
configuration data securely and safely.

Regardless of
which remote services your application connects to, you will likely need to
go through authentication, which involves sending over credentials such as
username and password or some other security token. This confidential
information must be stored somewhere close to your application securely
and safely. This chapter’s Secure Configuration pattern is about the best
ways to keep your credentials as secure as possible when running on
Kubernetes.

## Problem
Secrets are the Kubernetes answer for confidential configuration in-cluster storage.The most straightforward solution for secure configuration is decoding
encrypted information within the application itself. This approach always
works, and not just when running on Kubernetes. But it takes considerable
work to implement this within your code, and it couples your business logic
with this aspect of securing your configuration. There are better, more
transparent ways to do this on Kubernetes.

## Solution

The support for secure configuration on Kubernetes falls roughly into two
categorie:
### Out-of-cluster encryption
The gist of the out-of-cluster technique is simple: pick up secret and
confidential data from outside the cluster and transform it into a Kubernetes
Secret. A lot of projects have been grown that implement this technique.
This chapter looks at the three most prominent ones (as of 2023): Sealed
Secrets, External Secrets, and sops.


#### Sealed Secrets
One of the oldest Kubernetes add-ons for helping with encrypted secrets is
Sealed Secrets, introduced by Bitnami in 2017. The idea is to store the
encrypted data for a Secret in a CustomResourceDefinition (CRD)
SealedSecret. In the background, an operator monitors such resources and
creates one Kubernetes Secret for each SealedSecret with the decrypted
content.

It is important to
properly back up the secret key, as without it, it will not be possible to
decrypt the secrets if the operator is uninstalled. One potential drawback of
Sealed Secrets is that they require a server-side operator to be continuously
running in the cluster in order to perform the decryption.

#### External Secrets
The main difference between External
Secrets and Sealed Secrets is that you do not manage the encrypted data
storage yourself but rely on an external SMS to do the hard work, including
encryption, decryption, and secure persistence. That way, you benefit from
all the features of your cloud’s SMS, like key rotation and a dedicated user
interface. SMS also provides an excellent way of separating concerns so
that different roles can manage the application deployments and the secrets
separately.

#### Sops
Sops is not specific to
Kubernetes but allows you to encrypt and decrypt any YAML or JSON file
to safely store those in a source code repository. It does this by encrypting
all values of such a document but leaving the keys untouched.

Sops is an excellent solution for easy GitOps-style integration of Secrets
without worrying about installing and maintaining Kubernetes add-ons.
However, while your configuration can now be stored securely in Git, it is
essential to understand that as soon as those configurations have been
handed over to the cluster, anybody with elevated access rights can read that
data directly via the Kubernetes API.
If this is not something you can tolerate, we need to dig deeper into the
toolbox and look again at centralized SMSs。

### Centralized secret management, example: HashiCorp Vault
Besides baking individual secret handling into your application code, an
alternative is to keep the secure information outside the cluster in the
external SMS and request the confidential information on demand over
secure channels.

#### Secrets Store CSI Driver
The Container Storage Interface (CSI) is a Kubernetes API for exposing
storage systems to containerized applications. CSI shows the path for third-
party storage providers to plug in new types of storage that can be mounted
as volumes in Kubernetes. 

The Secrets Store CSI Driver supports the SMS from major cloud vendors
(AWS, Azure, and GCP) and HashiCorp Vault.
The Kubernetes setup for connecting a secret manager via the CSI driver
involves performing these two administrative tasks:
Installing the Secrets Store CSI Driver and configuration for accessing
a specific SMS. Cluster-admin permissions are required for the
installation process.
Configuring access rules and policies. Several provider-specific steps
need to be completed, but the result is that a Kubernetes service
account is mapped to a secret manager-specific role that allows access
to the secrets.

While the setup for a CSI Secret Storage drive is quite complex, the usage
is straightforward, and you can avoid storing confidential data within
Kubernetes. However, there are more moving parts than with Secrets alone,
so more things can go wrong, and it’s harder to troubleshoot.
Let’s look at a final alternative for offering secrets to applications via well-
known Kubernetes abstractions.

 **Pod injection**
 The CSI
abstraction for projecting secret information into volumes visible as files for
the deployed application is much more decoupled.
Alternative solutions leverage other well-known patterns described in this
book:Init Container，Sidecar。

You can leverage these patterns on your own for your applications, but this
is tedious. It is much better to let an external controller inject the init
container or sidecar into your application.

allows modification of any resource when it is created. When a Pod
specification contains a particular, vault-specific annotation, the vault
controller will modify this specification to add a container for syncing with
Vault and to mount a volume for the secret data.

While you still need to install the Vault Injector controller, it has fewer
moving parts than hooking up a CSI secret storage volume with the
provider deployment for a particular SMS product. 

## Discussion
Now that we have seen the many ways you can make access to your
confidential information more secure, the question is, which one is the best?

If your main goal is a simple way to encrypt Secrets stored in public-
readable places like a remote Git repository, the pure client-side
encryption that Sops offers is perfect。

The secret synchronization that the External Secrets Operator
implements is a good choice when separating the concerns of
retrieving credentials in a remote SMS and using them is essential.

The ephemeral volume projection of secret information provided by
Secret Storage CSI Providers is the right choice for you when you
want to ensure that no confidential information is stored permanently
in the cluster except the access tokens for accessing external vaults.
Sidebar injections like the Vault Sidecar Agent Injector have the
benefit of shielding from a direct access to an SMS. They are easily
approachable at the cost of blurring the boundary between developers and administrator because of security annotations leaking into
application deployment。

the techniques used (client-side encryption, Secret
synchronization, volume projections, and sidecar injections) are universal
and will be part of future solutions.






