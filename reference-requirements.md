# Reference Type Requirements

The ability to distribute and consume supply chain artifacts has driven a new set of requirements for adding information within an OCI Registry. [OCI Artifacts](https://github.com/opencontainers/artifacts/) enabled new, independent artifacts. [ORAS Artifacts](README.md) enables supply chain scenarios such as signatures, systems bill of materials (SBOM), security scan results and attestations.

The [ORAS Artifacts spec](https://github.com/oras-project/artifacts-spec/) accounts for these new reference types, however it does require support from registries to implement the new [artifact manifest](artifact-manifest.md), and the new [referrers api](manifest-referrers-api.md) to discover artifacts that refer to a given digest and/or tag.

To account for registries that have not yet implemented ORAS Artifacts, a fallback design will be provided. The assumption is the fallback will have some tradeoffs, as a fallback that implements the full set of requirements would question why a new manifest and referrers api would be required.

The following captures the requirements as a comparison for different implementations. The implementations should account for zero changes to registries that implement the [OCI Distribution-spec 1.1 ](https://github.com/opencontainers/distribution-spec/releases/tag/v1.0.1) to the full [ORAS Artifacts spec](README.md).

## Definitions

- **Artifact** - One conceptual piece of content stored as one or more blobs, represented by a Manifest. An artifact has a distinct lifecycle, represented by an associated tag.  
*(Examples: container images, wasm, helm charts)*
- **Reference Type** - One or more additional (detached) pieces of content, that enhances the content of referenced artifact without mutating referenced artifact. A reference type may contain blobs for larger content, but it may also be limited to signed annotations, providing attestations to a referenced artifact.  
*(Examples: signatures, SBoM, security scan results, policies, attestations)*
- **Subject** - Reference types are added to existing artifacts. As a reference is added, the artifact it references is call the **`subject`**.

## Requirements

To enable discussions for fallback design choices, the following lists the various requirements. Not all design options will serve all requirements. The goal is to serve as many critical requirements for existing and unchanged [OCI Distribution-spec 1.1 ](https://github.com/opencontainers/distribution-spec/releases/tag/v1.0.1) based registries, comparing to a full feature implementation.

The [best practices for consuming public content](https://opencontainers.org/posts/blog/2020-10-30-consuming-public-content/) involves promoting content across registries.
The list of requirements account for integration scenarios where an artifact, and the graph of its references may be promoted across [OCI Distribution-spec 1.1 ](https://github.com/opencontainers/distribution-spec/releases/tag/v1.0.1) based registries, with fallback support, to registries that support [ORAS Artifacts](README.md)

| # | Item | Description |
| - | - | - |
| 1 | Doesn't mutate the tag or digest of the subject artifact  | Adding a reference type doesn't change the manifest, or layers of the artifact being referenced. |
| 2 | Push a single level reference | Enable signature on an image, sbom on an image, but may not support signature on sbom for an image. |
| 2 | Push multi-level independent references | Enable a graph of artifacts, representing an image, signed sbom, signed scan result, with nested attestations. The graph has  _n_-depth support. (`image <--sbom <-- signature <-- receipt`). Each reference may be pushed independently, at different times. |
| 4 | Works with registries that enforce immutable tags | Some registries support locking tags from mutation. Pushing a graph of incremental references must not require an existing tag to be updated . |
| 4 | Each reference type is an independent entry | To support pulling individual reference artifacts, or promoting a filtered set of artifacts, each reference must exist independent from the other. |
| 5 | Supports multiple blobs as a reference type | An artifact may have multiple blobs that make up the artifact. A helm chart is a reference type, which has two layers. An SBOM may contain a collection of files. A scan result may contain an SBOM and a scan result, which may be persisted as two or more blobs |
| 6 | Supports annotations only | An attestation may be persisted as a signature with annotations. The signature and the attestations are stored as annotations on the manifest removing the need to persist and pull blobs |
| 6 | Lifecycle management, based on the subjects tag | Reference types are extensions to the artifact they reference. Deleting the root artifact must enable the registry the option to delete the graph of references |
| 7 | Non-impactful to existing container runtimes  | As registries are used to store multiple artifact types, existing container runtimes must not be impacted by accidental deployments. Issuing a deployment of a non-runtime-container based artifact must fail gracefully. The runtime must not attempt to download the artifact, enabling trojan horse style attacks. |
| 8 | Ability to pull a reference type by specifying the subject tag/digest and the `artifactType` | Reference types enable consumers to use the artifact reference embedded int e deployment file. A client must be capable of using this original tag or digest reference, and filter the reference types by an artifactType. For instance, return all the `application/vnd.cncf.notary.v2.signature`s  for `net-monitor:v1` |
| 9 | Ability to return a sorted list of references by date and artifactType | As registries store multiple scan results, clients will need a paged and ordered list of `scan-result/example` references to the `net-monitor:v1` image. This would enable pulling the first or most recent scan result |
| 10 | Ability to filter a list of references by an annotation | As registries are used to store multiple signatures or attestations, these may all be persisted as the same `artifactType`. To provide generic (non-artifact specific knowledge) referrer results, the registry must be capable of supporting filtering by `artifactType`, and named `annotations`. Example: `registry.acme-rockets.io/v2/net-monitor/_oras/artifacts/referrers?digest={$DIGEST}&artifactType={$ARTIFACTTYPE}&annotation={$ANNOTATION}` |
| 11 | Avoids race and contention conditions | Several registries support geo-replicated instances, where a reference type may be pushed from different nodes to different replicas. The design of independent reference artifacts, avoids contention and race condition as all individual artifacts have eventual consistency, where the registry manages the index of references |
| 12 | Supports references to OCI Indexes | A signature, attestation, or possibly platform aggregated SBOM may be associated with an OCI Index. While it's debateable if a single SBOM should be used to represent multiple architectures, as opposed to each architecture has its own SBOM, the design should be flexible for other reference types that may be more applicable. For example, [CNAB](https://cnab.io)s are independent artifacts that happen to use OCI index as their representation. |
| 13 | Doesn't impact existing tag listing results | A fallback may utilize a tag matching pattern. While is does solve some problems, it introduces others where users expect the tags to represent the primary artifacts, (container image, helm chart, wasm). The expectation a registry may implement patterned tag filtering is in the "registry code change" bucket. |

## Scenarios

List of scenarios for interoperability.

### Filtered Promotion

An artifact may have periodic scans and multiple attestations. As the artifact is built in the dev registry, it may be promoted through staging to production. It may be made public, for others to consume. At each stage, a set of reference types will exist, which may build up over time. As an artifact is promoted, the consumer may only care about the most recent scan result, or specific set of attestations. To promote these, each additional reference must be independent, allowing a filtered graph to be promoted. 

This makes using a single manifest to aggregate multiple references a challenge. If a single manifest (or OCI Index) is used to represent multiple artifacts, how is the subset promoted? Are new manifests/indexes expected to be re-created? If so, how does the graph of references maintain their digest reference?

### Local signatures coexist with upstream signatures that are periodically synced

## Comparison of Options

A matrix of designs that support each requirement. The options fall into two buckets:

1. Zero changes to an existing [OCI Distribution-spec 1.1 ](https://github.com/opencontainers/distribution-spec/releases/tag/v1.0.1) based registry.
2. Any expectation of a change, from a tag list filtering to new manifest support or a [referrers api](manifest-referrers-api.md).

The choice of options should be considered a weighted scale, where zero changes are easy, and any changes are hard. Once enhancements are added, the size of the enhancement is non-linear, when comparing to zero. Adding any registry, server-side changes is the largest investment.

| # | Item | Fallback 1 | Fallback 2 | Updates to OCI<br>Manifest/Index Specs | ORAS<br> Artifact manifest spec|
| - | - | - | - | - | - |
| 0 | Works with [OCI Distribution-spec 1.1 ](https://github.com/opencontainers/distribution-spec/releases/tag/v1.0.1) based registries |  :heavy_check_mark: | :heavy_check_mark: | :x:  | :x: |
| 1 | Doesn't mutate the tag or digest of the subject artifact  |  :grey_question: | :grey_question: | :grey_question: | :heavy_check_mark: |
| 2 | Push a single level reference |  :grey_question: | :grey_question: | :grey_question: | :heavy_check_mark: |
| 2 | Push multi-level independent references |  :grey_question: | :grey_question: | :grey_question: | :heavy_check_mark: |
| 4 | Works with registries that enforce immutable tags |  :grey_question: | :grey_question: | :grey_question: | :heavy_check_mark: |
| 4 | Each reference type is an independent entry |  :grey_question: | :grey_question: | :grey_question: | :heavy_check_mark: |
| 5 | Supports multiple blobs as a reference type |  :grey_question: | :grey_question: | :grey_question: | :heavy_check_mark: |
| 6 | Supports annotations only |  :grey_question: | :grey_question: | :grey_question: | :heavy_check_mark: |
| 6 | Lifecycle management, based on the subjects tag |  :grey_question: | :grey_question: | :grey_question: | :heavy_check_mark: |
| 7 | Non-impactful to existing container runtimes  |  :grey_question: | :grey_question: | :grey_question: | :heavy_check_mark: |
| 8 | Ability to pull a reference type by specifying the subject tag/digest and the `artifactType` |  :grey_question: | :grey_question: | :grey_question: | :heavy_check_mark: |
| 9 | Ability to return a sorted list of references by date and artifactType |  :grey_question: | :grey_question: | :grey_question: | :heavy_check_mark: |
| 10 | Ability to filter a list of references by an annotation |  :grey_question: | :grey_question: | :grey_question: | :heavy_check_mark: |
| 11 | Avoids race and contention conditions |  :grey_question: | :grey_question: | :grey_question: | :heavy_check_mark: |
| 12 | Supports references to OCI Indexes |  :grey_question: | :grey_question: | :grey_question: | :heavy_check_mark: |
| 13 | Doesn't impact existing tag listing results |  :grey_question: | :grey_question: | :grey_question: | :heavy_check_mark: |

### Key:

- :heavy_check_mark: : supported
- :white_check_mark: : requires workarounds to support
- :grey_question: : research needed 
- :x: : unsupported