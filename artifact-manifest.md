# ORAS Artifact Manifest Spec

The ORAS Artifact manifest is similar to the [OCI image manifest][oci-image-manifest-spec], but removes constraints defined on the image-manifest such as a required `config` object and required & ordinal `layers`.
It then adds a `subject` property supporting a graph of independent, but link artifacts.
The addition of a new manifest does not change, nor impact the `image.manifest`.
It provides a means to define a wide range of artifacts, including a chain of related artifacts enabling SBoMs, on-demand loading, signatures and metadata that can be related to an `image.manifest`, `image.index` or another `artifact.manifest`.
By defining a new manifest, registries and clients opt-into new capabilities, without breaking existing registry and client behavior or setting expectations for scenarios to function when the client and/or registry may not yet implement new capabilities.

This section defines the `application/vnd.cncf.oras.artifact.manifest.v1+json` media type.

## ORAS Artifact Manifest Properties

The `artifact.manifest` provides an optional collection of `blobs`, an optional reference to the manifest of another artifact and an `artifactType` to differentiate different types of artifacts (such as signatures, sboms and security scan results)

- **`mediaType`** *string*

  This field contains the `mediaType` of this document, differentiating from [image-manifest][oci-image-manifest-spec] and [image-index][oci-image-index]. The `mediaType` for this manifest type MUST be `application/vnd.cncf.oras.artifact.manifest.v1+json`, where the version WILL change to reflect newer versions.
   
- **`artifactType`** *string*

  The REQUIRED `artifactType` is a unique value, as registered with [iana.org][registering-iana].
  The `artifactType` values are equivalent to the values used in the `manifest.config.mediaType` in [OCI Artifacts][oci-artifacts].
  Examples include `sbom/example`, `application/vnd.cncf.notary.v2`.
  For details on creating a unique `artifactType`, see [OCI Artifact Authors Guidance][oci-artifact-authors]

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

### Example ORAS Artifact Manifests

- [`net-monitor:v1` oci container image](./examples/net-monitor-oci-image.json)
- [`net-monitor:v1` notary v2 signature](./examples/net-monitor-image-signature.json)
- [`net-monitor:v1` sample sbom](./examples/net-monitor-image-sbom.json)
- [`net-monitor:v1` nydus image with on-demand loading](./examples/net-monitor-image-nydus-ondemand-loading.json)

## Push Validation

Following the [distribution-spec push api](https://github.com/opencontainers/distribution-spec/blob/main/spec.md#push), all `blobs` *and* the `subject` descriptors SHOULD exist when pushed to a distribution instance.

## Lifecycle Management

Registries MAY treat the lifecycle of a reference type object, such as an SBoM or signature, as being tied to its `subject`. In such registries, when the `subject` is deleted or marked for garbage collection, the defined artifact is subject to deletion as well, unless the artifact is tagged.

## Further Reading

- [Usage and Scenarios](./scenarios.md)
- [Comparing the ORAS Artifact Manifest and OCI Image Manifest][manifest-differences]
- [Referrers API](./manifest-referrers-api.md) for more information on listing references

[oci-artifacts]:                   https://github.com/opencontainers/artifacts
[oci-artifact-authors]:            https://github.com/opencontainers/artifacts/blob/master/artifact-authors.md
[oci-image-manifest-spec]:         https://github.com/opencontainers/image-spec/blob/master/manifest.md
[oci-image-manifest-spec-layers]:  https://github.com/opencontainers/image-spec/blob/master/manifest.md#image-manifest-property-descriptions
[oci-image-index]:                 https://github.com/opencontainers/image-spec/blob/master/image-index.md
[oci-distribution-spec]:           https://github.com/opencontainers/distribution-spec
[registering-iana]:                https://github.com/opencontainers/artifacts/blob/master/artifact-authors.md#registering-unique-types-with-iana
[descriptor]:                      ./descriptor.md
[manifest-differences]: ./README.md#comparing-the-oras-artifact-manifest-and-oci-image-manifest
