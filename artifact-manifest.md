# ORAS Artifact Manifest Spec

The ORAS Artifact manifest is similar to the [OCI image manifest][oci-image-manifest-spec], but removes constraints defined on the image-manifest such as a required `config` object and required & ordinal `layers`.
It then adds a `subject` property supporting a graph of independent, but link artifacts.
The addition of a new manifest does not change, nor impact the `image.manifest`. It provides a means to define a wide range of artifacts, including a chain of related artifacts enabling SBoMs, on-demand loading, signatures and metadata that can be related to an `image.manifest` or `image.index`.
By defining a new manifest, registries and clients opt-into new capabilities, without breaking existing registry and client behavior or setting expectations for scenarios to function when the client and/or registry may not yet implement new capabilities.

For usage and scenarios, see [scenarios.md](./scenarios.md)

## ORAS Artifact and Image Spec Differences

The high-level differences with the `oras.artifact.manifest` and the `oci.image.manifest`:

| OCI Image Manifest | ORAS Artifacts Manifest |
|-|-|
| `config` REQUIRED | `config` OPTIONAL as it's just another entry in the `blobs` collection with a config `mediaType` |
| `layers` REQUIRED | `blobs` are OPTIONAL, which were renamed from `layers` to reflect general usage |
| `layers` ORDINAL | `blobs` are defined by the specific artifact spec. For example, Helm utilizes two independent, non-ordinal blobs, while other artifact types like container images may require blobs to be ordinal |
| `manifest.config.mediaType` used to uniquely identify artifact types. | `manifest.artifactType` added to lift the workaround for using `manifest.config.mediaType` on a REQUIRED, but not always used `config` property. Decoupling `config.mediaType` from `artifactType` enables artifacts to OPTIONALLY share config schemas. |
| | `subject` OPTIONAL, enabling an artifact to extend another artifact (SBOM, Signatures, Nydus, Scan Results)
| | `/referrers` api for discovering referenced artifacts, with the ability to filter by `artifactType` |
| | Lifecycle management defined, starting to provide standard expectations for how users can manage their content |

### Example ORAS Artifact Manifests

- [`net-monitor:v1` oci container image](./examples/net-monitor-oci-image.json)
- [`net-monitor:v1` notary v2 signature](./examples/net-monitor-image-signature.json)
- [`net-monitor:v1` sample sbom](./examples/net-monitor-image-sbom.json)
- [`net-monitor:v1` nydus image with on-demand loading](./examples/net-monitor-image-nydus-ondemand-loading.json)

## ORAS Artifact Manifest Properties

The `artifact.manifest` provides an optional collection of `blobs`, an optional reference to the manifest of another artifact and an `artifactType` to differentiate different types of artifacts (such as signatures, sboms and security scan results)

- **`mediaType`** *string*

  This property is reserved for use, to maintain compatibility. When used, this field contains the `mediaType` of this document, differentiating from [image-manifest][oci-image-manifest-spec] and [oci-image-index]. The `mediaType` for this manifest type MUST be `application/vnd.cncf.oras.artifact.manifest.v1+json`, where the version WILL change to reflect newer versions. Artifact authors SHOULD support multiple `mediaType` versions to provide the best user experience for their artifact type.
   
- **`artifactType`** *string*

  The REQUIRED `artifactType` is unique value, as registered with iana.org. See [registering unique types.][registering-iana]. The `artifactType` is equivalent to ORAS Artifacts that used the `manifest.config.mediaType` to differentiate the type of artifact. Artifact authors that implement `oras.artifact.manifest` use `artifactType` to differentiate the type of artifact. example:(`application/x.example.sbom.v0` from `application/vnd.cncf.notary.v2`).

- **`blobs`** *array of objects*

    An OPTIONAL collection of 0 or more blobs. The blobs array is analogous to [oci.image.manifest layers][oci-image-manifest-spec-layers], however unlike [image-manifest][oci-image-manifest-spec], the ordering of blobs is specific to the artifact type. Some artifacts may choose an overlay of files, while other artifact types may store independent collections of files.

    - Each item in the array MUST be an [artifact descriptor][descriptor], and MUST NOT refer to another `manifest` providing dependency closure.
    - The max number of blobs is not defined, but MAY be limited by [distribution-spec][oci-distribution-spec] implementations.
    - An encountered `[blobs].descriptor.mediaType` that is unknown to the implementation MUST be ignored.

- **`subject`** *descriptor*

   An OPTIONAL reference to any existing manifest within the repository. When specified, the artifact is said to be dependent upon the referenced `subject`.
   - The item MUST be an [artifact descriptor][descriptor] representing a manifest. Descriptors to blobs are not supported. The registry MUST return a `400` response code when `subject` is not found in the same repository, and not a manifest.

- **`annotations`** *string-string map*

    This OPTIONAL property contains arbitrary metadata for the artifact manifest.
    This OPTIONAL property MUST use the [annotation rules](annotations.md#rules).

## Push Validation

Following the [distribution-spec push api](https://github.com/opencontainers/distribution-spec/blob/main/spec.md#push), all `blobs` *and* the `subject` descriptors SHOULD exist when pushed to a distribution instance.

## Lifecycle Management

Registries MAY treat the lifecycle of a reference type object, such as an SBoM or signature, as being tied to its `subject`. In such registries, when the `subject` is deleted or marked for garbage collection, the defined artifact is subject to deletion as well, unless the artifact is tagged.

## Further Reading

- [Scenarios](./scenarios.md)
- [Referrers API](./manifest-referrers-api.md) for more information on listing references

[oci-image-manifest-spec]:         https://github.com/opencontainers/image-spec/blob/master/manifest.md
[oci-image-manifest-spec-layers]:  https://github.com/opencontainers/image-spec/blob/master/manifest.md#image-manifest-property-descriptions
[oci-image-index]:                 https://github.com/opencontainers/image-spec/blob/master/image-index.md
[oci-distribution-spec]:           https://github.com/opencontainers/distribution-spec
[registering-iana]:                ./artifact-authors.md#registering-unique-types-with-iana
[descriptor]:                      ./descriptor.md
