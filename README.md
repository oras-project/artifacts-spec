# ORAS Artifacts Specification

[OCI Artifacts][oci-artifacts] generalized the ability to persist artifacts within an [OCI Distribution conformant][oci-conformance] registry. The majority of cloud registries, products and projects support pushing and pulling OCI Artifacts to a registry enabling users to benefit from the performance, security, reliability capabilities. Thus avoiding the need to run, manage or care for **Y**et **A**nother **S**torage **S**ervice (YASS).

## How does ORAS Artifacts relate to OCI Artifacts?

OCI Artifacts defines how to implement stand-alone artifacts that can fit within the constraints of the image-spec. ORAS Artifacts uses the `manifest.config.mediaType` to identify the artifact is something other than a container image. While this validated the ability to generalize the **C**ontent **A**ddressable **S**torage (CAS) capabilities of [OCI Distribution][oci-distribution], a new set of artifacts require additional capabilities that aren't constrained to the image-spec. ORAS Artifacts provide a more generic means to store a wider range of artifact types, including references between artifacts.
For more info, see: [Discussion of a new manifest #41](https://github.com/opencontainers/artifacts/discussions/41)

## Table of Contents:

- [Overview](#overview)
- [Project Status](#project-status)
- [ORAS Artifacts Manifest Overview][artifact-manifest]
- [ORAS Artifacts Manifest Spec][artifact-manifest-spec]
- [ORAS Artifacts Referrers Spec][artifact-referrers-spec]
- [CNCF Distribution Support for ORAS Artifacts][cncf-distribution-reftypes]
- [ORAS experimental support for oras.artifact.manifest references][oras-artifacts] to `push`, `discover`, `pull` referenced artifact types.
- [Code of Conduct](#code-of-conduct)

## Overview

As the distribution of secure supply chain content becomes a primary focus, users and registry operators are looking to extend the capabilities for storing artifacts including content signing, SBoMs, artifact security scan results. To provide these capabilities, the ORAS Artifacts Spec will provide a specification for storing a broad range of types, including the ability to store references between types, enabling a graph of objects that registry operators and client can logically reason about.

![](media/net-monitor-graph.svg)

The ORAS Artifacts specs will build upon the [OCI distribution-spec][oci-distribution] assuring registry operators can opt-into the behavior, ensuring users and clients have well understood expectations for the lifecycle management capabilities for storing artifacts and the references between artifacts.


The artifact manifest approach to reference types is based on a new manifest, enabling registries and clients to opt-into the behavior, with clear and consistent expectations, rather than slipping new content into a registry, or client, that may, or may not know how to lifecycle manage the new content.

| Existing Image Manifest | Proposed Artifacts Manifest |
|-|-|
| `config` REQUIRED | `config` optional as it's just another entry in the `blobs` collection with a `config mediaType` |
| `layers` REQUIRED | `blobs`, which renamed `layers` to reflect general usage are OPTIONAL |
| `layers` ORDINAL | `blobs` are defined by the specific artifact spec. Helm isn't ordinal, while other artifact types, like container images MAY make them ordinal |
| `manifest.config.mediaType` used to uniquely identify different artifact types. | `manifest.artifactType` added to lift the workaround for using `manifest.config.mediaType` on a REQUIRED, but not always used property, decoupling `config.mediaType` from `artifactType`. |
| | `subjectManifest` OPTIONAL, enabling an artifact to extend another artifact (SBOM, Signatures, Nydus, Scan Results, )
| | `/referrers` api for discovering referenced artifacts, with the ability to filter by `artifactType` |
| | Lifecycle management defined, starting to provide standard expectations for how users can manage their content. It doesn't define GC as an internal detail|

## Project status

The ORAS artifacts-spec is experimental with the goal of providing a working implementation of the [OCI reference types proposal][oci-reference-types-proposal]. The intent is that once sufficiently proven it will be presented to OCI TOB for recommendation to be part of the specifications under their governance.

This decision was made with the OCI TOB during the [weekly discussion][oci-tob-weekly-discussion] on July 21, 2021 while they work on creating a process to incubate new work under the [OCI working group][oci-working-group-proposal].

## Community

- Slack: [#oras-artifacts-spec](https://cloud-native.slack.com/archives/C02AJS1BUTX)
  - To participate in this channel, join CNCF slack at https://slack.cncf.io/

## Code of Conduct

This project has adopted the [CNCF Code of Conduct](https://github.com/cncf/foundation/blob/master/code-of-conduct.md). See [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) for further details.

[oci-artifacts]:                    https://github.com/opencontainers/artifacts
[oci-conformance]:                  https://github.com/opencontainers/oci-conformance/tree/main/distribution-spec
[oci-distribution]:                 https://github.com/opencontainers/distribution-spec
[cncf-distribution-reftypes]:       https://github.com/oras-project/distribution/
[artifact-manifest]:                ./scenarios.md
[artifact-manifest-spec]:           ./artifact-manifest.md
[artifact-referrers-spec]:          ./manifest-referrers-api.md
[oras-artifacts]:                   https://github.com/oras-project/oras
[oci-reference-types-proposal]:     https://github.com/opencontainers/artifacts/pull/29
[oci-tob-weekly-discussion]:        https://hackmd.io/El8Dd2xrTlCaCG59ns5cwg#July-21-2021
[oci-working-group-proposal]:       https://github.com/opencontainers/tob/pull/99
