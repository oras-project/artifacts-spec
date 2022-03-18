# Manifest Referrers API

[Artifact-manifest](./artifact-manifest.md) provides the ability to reference artifacts to existing artifacts.
Reference artifacts include signatures, SBoMs and many other types.
Artifacts that reference other artifacts SHOULD NOT be tagged, as they are considered enhancements to the artifacts they reference.
The `referrers` extension API is provided to discover these artifacts.
An artifact client would parse the returned [artifact descriptors][descriptor], determining which  artifact manifest they will pull and process.

The `referrers` API returns all artifacts that have a `subject` of the given manifest digest.
Reference artifact requests are scoped to a repository, ensuring access rights for the repository can be used as authorization for the referenced artifacts.

Artifact references are defined in the [artifact-manifest][oras.artifact.manifest-spec] spec through the [`subject`][oras.artifact.manifest-spec-manifests] property.

## API Discovery

API discovery follows the  [OCI extensions specification][distribution-extension].
Clients can check for the support of the `referrers` API by making a
GET request to the OCI extensions discovery endpoint under a respository as
shown below.

```http
GET /v2/{repository}/_oci/ext/discover
```

The reponse SHOULD contain an extension with the name of `cncf.oras.referrers`
and the `url` path where the referrers can be requested.

```http
200 OK
Content-Length: <length>
Content-Type: application/json

{
    "extensions": [
        {
            "name": "cncf.oras.referrers",
            "description": "ORAS referrers listing API",
            "url": "_oras/artifacts/referrers"
        }
    ]
}
```

## API Path

The `referrers` api are provided on the [distribution-spec][oci-distribution-spec] paths as described below.
Pathing of the referrers api provides consistent namespace/repository paths, enabling registry operators to implement consistent auth access, using existing tokens for content.

**template:**

```rest
GET /v2/{repository}/_oras/artifacts/referrers?digest={digest}
```

**expanded example:**

```rest
GET /v2/net-monitor/_oras/artifacts/referrers?digest=sha256:3c3a4604a545cdc127456d94e421cd355bca5b528f4a9c1905b15da2eb4a4c6b
```

## Versioning

This specification defines the versioning behavior of `_oras/artifacts/referrers` API.
Implementations SHOULD return `ORAS-Api-Version: oras/1.0`.  Clients MUST account
for version checking as future major versions MAY NOT be compatible. Future minor
versions MUST be additive.

```http
GET /v2/<repository>/_oras/artifacts/referrers?digest={digest}
...
ORAS-Api-Version: oras/1.0
```

## Artifact Referrers API results

- Implementations MUST implement [paging](#paging-results).
- Implementations MUST implement [sorting](#sorting-results)
- Implementations SHOULD implement [`artifactType` filtering](#filtering-results).

Some artifacts types including signatures, may return multiple signatures of the same `artifactType`.
For cases where multiple artifacts are returned to the client, it may be necessary to pull each artifact's manifest in order to determine whether or not the full artifact is needed.
Maintainers of the standards utilizing references SHOULD define standard sets of annotations that will allow clients to determine whether or not each artifact needs to be downloaded in full.

While this will cause additional round trips, manifests are typically small in comparison to the full pull time for a manifest and its blobs or layers.
In future versioned releases, responses MAY be extended to include a `data` field representing the `base64` encoded manifest blob.

This paged result MUST return the following elements:

- `referrers`: A list of [artifact descriptors][descriptor] that reference the
given manifest. The list MUST include these references even if the given
manifest does not exist in the repository. The list MUST be empty
if there are no artifacts referencing the given manifest.

**example result of artifacts that reference the `net-monitor` image:**

```json
{
  "referrers": [
    {
      "digest": "sha256:3c3a4604a545cdc127456d94e421cd355bca5b528f4a9c1905b15da2eb4a4c6b",
      "mediaType": "application/vnd.cncf.oras.artifact.manifest.v1+json",
      "artifactType": "signature/example",
      "size": 312
    },
    {
      "digest": "sha256:3c3a4604a545cdc127456d94e421cd355bca5b528f4a9c1905b15da2eb4a4c6b",
      "mediaType": "application/vnd.cncf.oras.artifact.manifest.v1+json",
      "artifactType": "sbom/example",
      "size": 237
    }
  ]
}
```

**example result for a manifest that has no artifacts referencing it:**

```json
{
  "referrers": []
}
```

### Paging Results

The `referrers` API MUST provide for paging, returning a list of [artifact descriptors](./descriptor.md).
Page size can be specified by adding a `n` parameter to the request URL, indicating that the response should be limited to `n` results.

- If specified, servers MAY return up to `n` items from the entire result set.
- When `n` is not provided, servers MAY return a default number of items, which may be implementation specific.

A paginated flow begins as:

**template:**

```rest
GET /v2/{repository}/_oras/artifacts/referrers?digest={digest}&n=<integer>
```

**expanded example:**

```rest
GET /v2/{repository}/_oras/artifacts/referrers?digest=sha256:3c3a4604a545cdc127456d94e421cd355bca5b528f4a9c1905b15da2eb4a4c6b&n=10
```

The above specifies that a referrers response should be returned limiting the number of results to `n`.
The response to such a request would look as follows:

```json
200 OK
ORAS-Api-Version:oras/1.0
Link: <url>; rel="next"

{
  "referrers": [
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

- The presence of the `Link` header communicates to the client that the server has more items. Clients are expected
  to follow the link to fetch the next page of items, irrespective of the number of items received in the current
  response.
- If the header is not present, clients can assume that all items have been received.

> NOTE: In the request template above, the brackets around the url are required.

For example, if the url is:

```
http://example.com/v2/hello-world/_oras/artifacts/referrers?digest=sha256:3c3a4604a545cdc127456d94e421cd355bca5b528f4a9c1905b15da2eb4a4c6b&n=5&nextToken=abc
```

The value of the header would be:

```
<http://example.com/v2/hello-world/_oras/artifacts/referrers?digest=sha256:3c3a4604a545cdc127456d94e421cd355bca5b528f4a9c1905b15da2eb4a4c6b&n=5&nextToken=abc>; rel="next"`.
```

Please see [RFC5988][rfc5988] for details.

### Sorting Results
The `/referrers` API MUST allow for artifacts to be sorted by the date and time in which they were created, which SHOULD be included in the artifact manifest's list of `annotations`.
The artifact's creation time MUST be the value of the `io.cncf.oras.artifact.created` annotation, as specified in the [artifact-manifest spec][artifact-manifest-spec].
The results of the `/referrers` API MUST list artifacts that were created more recently first.
Artifacts that do not have the `io.cncf.oras.artifact.created` annotation MUST appear after those with creation times specified in the list of results.
There is no specified ordering for artifacts that do not include the creation time in their list of `annotations`.

### Filtering Results

The `referrers` API MAY provide for filtering of `artifactTypes`.
Artifact clients MUST account for implementations that MAY NOT support filtering.
Artifact clients MUST revert to client side filtering to determine which `artifactTypes` they will process.

Request referenced artifacts by `artifactType`

**template:**

```rest
GET /v2/{repository}/_oras/artifacts/referrers?digest={digest}&n=10&artifactType={artifactType}
```

**expanded example:**

```rest
GET /v2/net-monitor/_oras/artifacts/referrers?digest=sha256:3c3a4604a545cdc127456d94e421cd355bca5b528f4a9c1905b15da2eb4a4c6b&n=10&artifactType=signature%2Fexample
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
[distribution-extension]:                https://github.com/opencontainers/distribution-spec/tree/main/extensions
