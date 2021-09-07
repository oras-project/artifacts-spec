# Manifest Referrers API

[Artifact-manifest](./artifact-manifest.md) provides the ability to reference artifacts to existing artifacts.
Reference artifacts include Notary v2 signatures, SBoMs and many other types.
Artifacts that reference other artifacts SHOULD NOT be tagged, as they are considered enhancements to the artifacts they reference.
To discover referenced artifacts a manifest `referrers/` API is provided.
An artifact client, such as a Notary v2 client would parse the returned [artifact descriptors][descriptor], determining which `artifact.manifest` they will pull and process.

The `referrers/` API returns all artifacts that have a `subject` to given manifest digest.
Referenced artifact requests are scoped to a repository, ensuring access rights for the repository can be used as authorization for the referenced artifacts.

Artifact references are defined in the [artifact-manifest][oras.artifact.manifest-spec] spec through the [`subject`][oras.artifact.manifest-spec-manifests] property.

## API Path

The `referrers/` api is sits alongside the [distribution-spec][oci-distribution-spec] paths avoiding conflict with existing or new distribution spec apis.
Pathing within the referrers api provides consistent namespace/repo paths, enabling registry operators to implement consistent auth access, using existing tokens for content.

**template:**

```rest
GET /oras/artifacts/v1/{repository}/manifests/{digest}/referrers
```

**expanded example:**

```rest
GET /oras/artifacts/v1/net-monitor/manifests/sha256:3c3a4604a545cdc127456d94e421cd355bca5b528f4a9c1905b15da2eb4a4c6b/referrers
```

## Versioning

This spec defines the behavior of the `/oras/artifacts/v1` version. Clients MUST account for version checking as future major versions MAY NOT be compatible.
Future Minor versions MUST be additive.

## Artifact Referrers API results

- Implementations MUST implement [paging](#paging-results).
- Implementations MAY implement [`artifactType` filtering](#filtering-results).

Some artifacts types including Notary v2 signatures, may return multiple signatures of the same `artifactType`.
For cases where multiple artifacts are returned to the client, it may be necessary to pull each artifact's manifest in order to determine whether or not the full artifact is needed.
Maintainers of the standards utilizing references SHOULD define standard sets of annotations that will allow clients to determine whether or not each artifact needs to be downloaded in full.

While this will cause additional round trips, manifests are typically small in comparison to the full pull time for a manifest and its blobs or layers.
In future versioned releases, responses MAY be extended to include a `data` field representing the `base64` encoded manifest blob.

This paged result MUST return the following elements:

- `references`: A list of [artifact descriptors][descriptor] that reference the given `subject`.

**example result of artifacts that reference the `net-monitor` image:**
```json
{
  "references": [
    {
      "digest": "sha256:3c3a4604a545cdc127456d94e421cd355bca5b528f4a9c1905b15da2eb4a4c6b",
      "mediaType": "application/vnd.cncf.oras.artifact.manifest.v1+json",
      "artifactType": "application/vnd.cncf.notary.v2",
      "size": 312
    },
    {
      "digest": "sha256:3c3a4604a545cdc127456d94e421cd355bca5b528f4a9c1905b15da2eb4a4c6b",
      "mediaType": "application/vnd.cncf.oras.artifact.manifest.v1+json",
      "artifactType": "application/x.example.sbom.v0",
      "size": 237
    }
  ]
}
```

### Paging Results

The `/referrers` API MUST provide for paging, returning a list of [artifact descriptors](./descriptor.md).
Page size can be specified by adding a `n` parameter to the request URL, indicating that the response should be limited to `n` results.

* If specified, servers MAY return up to `n` items from the entire result set.
* When `n` is not provided, servers MAY return a default number of items, which may be implementation specific.

A paginated flow begins as:

**template:**

```rest
GET /oras/artifacts/v1/{repository}/manifests/{digest}/referrers?n=<integer>
```

**expanded example:**

```rest
GET /oras/artifacts/v1/net-monitor/manifests/sha256:3c3a4604a545cdc127456d94e421cd355bca5b528f4a9c1905b15da2eb4a4c6b/referrers?n=10
```

The above specifies that a referrers response should be returned limiting the number of results to `n`. There is no
ordering imposed on the resulting collection. The response to such a request would look as follows:

```json
200 OK
Link: <url>; rel="next"

{
  "references": [
    {
      "digest": "<string>",
      "mediaType": "<string>",
      "artifactType": "<string>",
      "size": <integer>
    },
    ...
  ]
}
```

The above includes up to `n` items from the result set. If there are more items, the URL for the next collection is
encoded in a [RFC5988][rfc5988] `Link` header, as a "next" relation. Clients SHOULD treat this as an opaque value and not try to
construct it themselves.

* The presence of the `Link` header communicates to the client that the server has more items. Clients are expected
  to follow the link to fetch the next page of items, irrespective of the number of items received in the current
  response.
* If the header is not present, clients can assume that all items have been received.

> NOTE: In the request template above, the brackets around the url are required. 

For example, if the url is:
```
http://example.com/oras/artifacts/v1/hello-world/manifests/sha256:3c3a4604a545cdc127456d94e421cd355bca5b528f4a9c1905b15da2eb4a4c6b/referrers?n=5&nextToken=abc
```
The value of the header would be:
```
<http://example.com/oras/artifacts/v1/hello-world/manifests/sha256:3c3a4604a545cdc127456d94e421cd355bca5b528f4a9c1905b15da2eb4a4c6b/referrers?n=5&nextToken=abc>; rel="next"`.
```
Please see [RFC5988][rfc5988] for details.

### Filtering Results

The `/referrers` API MAY provide for filtering of `artifactTypes`.
Artifact clients MUST account for implementations that MAY NOT support filtering.
Artifact clients MUST revert to client side filtering to determine which `artifactTypes` they will process.

Request referenced artifacts by `artifactType`

**template:**
```rest
GET /oras/artifacts/v1/{repository}/manifests/{digest}/referrers?n=10&artifactType={artifactType}
```

**expanded example:**

```rest
GET /oras/artifacts/v1/net-monitor/manifests/sha256:3c3a4604a545cdc127456d94e421cd355bca5b528f4a9c1905b15da2eb4a4c6b/referrers?n=10&artifactType=application%2Fvnd.org.cncf.notary.v2
```

## Further Reading

- [Scenarios](./scenarios.md)
- [artifact-manifest spec][artifact-manifest-spec]

[artifact-manifest-spec]:                ./artifact-manifest.md
[descriptor]:                            ./descriptor.md
[oras.artifact.manifest-spec]:           ./artifact-manifest.md
[oras.artifact.manifest-spec-manifests]: ./artifact-manifest.md#oras-artifact-manifest-properties
[oci-distribution-spec]:                 https://github.com/opencontainers/distribution-spec
[rfc5988]:                               https://datatracker.ietf.org/doc/html/rfc5988