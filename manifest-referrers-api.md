# Manifest Referrers API

[Artifact-manifest](./artifact-manifest.md) provides the ability to reference artifacts to existing artifacts. Reference artifacts include Notary v2 signatures, SBoMs and many other types. Artifacts that reference other artifacts SHOULD NOT be tagged, as they are considered enhancements to the artifacts they reference. To discover referenced artifacts a manifest referrers API is provided. An artifact client, such as a Notary v2 client would parse the returned manifest descriptors, determining which manifest type they will pull and process.

The `referrers` API returns all artifacts that have a `subjectManifest` to given manifest digest. Referenced artifact requests are scoped to a repository, ensuring access rights for the repository can be used as authorization for the referenced artifacts.

Artifact references are defined in the [artifact.manifest spec][oras.artifact.manifest-spec] through the [`subjectManifest`][oras.artifact.manifest-spec-manifests] property.

## Request All Artifact References

The referrers api is sits alongside the [distribution-spec][oci-distribution-spec] paths avoiding any conflict with existing or new distribution apis. Pathing within the referrers api provides consistent repo/namespace paths, enabling registry operators to implement consistent auth access, using existing tokens for content.

This spec defines the behavior of the `v1` version. Clients MUST account for version checking as future major versions MAY NOT be compatible. Future Minor versions MUST be additive.

The `/referrers` API MUST provide for paging. The default page size SHOULD be set to 10.

```rest
GET /oras/artifacts/v1/{repository}/manifests/{digest}/referrers?n=10
```

**expanded example:**

```rest
GET /oras/artifacts/v1/net-monitor/manifests/sha256:3c3a4604a545cdc127456d94e421cd355bca5b528f4a9c1905b15da2eb4a4c6b/referrers?n=10
```

The `/referrers` API MAY provide for filtering of `artifactTypes`. Artifact clients MUST account for [distribution-spec][oci-distribution-spec] implementations that MAY NOT support filtering. Artifact clients MUST revert to client side filtering to determine which `artifactTypes` they will process.

### Request Artifacts of a specific media type

**template:**
```rest
GET /oras/artifacts/v1/{repository}/manifests/{digest}/referrers?n=10&artifactType={artifactType}
```

**expanded example:**

```rest
GET /oras/artifacts/v1/net-monitor/manifests/sha256:3c3a4604a545cdc127456d94e421cd355bca5b528f4a9c1905b15da2eb4a4c6b/referrers?n=10&artifactType=org.cncf.notary.v2
```

### Artifact Referrers API results

[distribution-spec][oci-distribution-spec] implementations MAY implement `artifactType` filtering. Some artifacts types including Notary v2 signatures, may return multiple signatures of the same `artifactType`. For cases where multiple artifacts are returned to the client, it may be necessary to pull each artifact's manifest in order to determine whether or not the full artifact is needed. Maintainers of the standards utilizing references SHOULD define standard sets of annotations that will allow clients to determine whether or not each artifact needs to be downloaded in full.

While this will cause additional round trips, manifests are typically small in comparison to the full pull time for a manifest and its blobs or layers. In the future, responses could be extended to include a `data` field representing the base64 encoded manifest blob.

This paged result MUST return the following elements:

- `referrers`: The list of `reference descriptors` that reference the given object. The descriptors used in this API are defined in greater detail [here](descriptor.md).
- `@nextLink`: Used for paged results.

As an example, Notary v2 manifests use annotations to determine which Notary v2 signature they should retrieve: `"org.cncf.notary.v2.signature.subject": "wabbit-networks.io"`

**example result of artifacts that reference the `net-monitor` image:**
```json
{
  "references": [
    {
      "digest": "sha256:3c3a4604a545cdc127456d94e421cd355bca5b528f4a9c1905b15da2eb4a4c6b",
      "mediaType": "application/vnd.cncf.oras.artifact.manifest.v1+json",
      "artifactType": "cncf.notary.v2",
      "size": 312
    },
    {
      "digest": "sha256:3c3a4604a545cdc127456d94e421cd355bca5b528f4a9c1905b15da2eb4a4c6b",
      "mediaType": "application/vnd.cncf.oras.artifact.manifest.v1+json",
      "artifactType": "example.sbom.v0",
      "size": 237
    }
  ],
  "@nextLink": "{opaqueUrl}"
}
```

[oras.artifact.manifest-spec]:           ./artifact-manifest.md
[oras.artifact.manifest-spec-manifests]: ./artifact-manifest.md#oras-artifact-manifest-properties
[oci-distribution-spec]:                 https://github.com/opencontainers/distribution-spec
