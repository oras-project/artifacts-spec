# Getting Started with ORAS Artifacts

A quick-start for push, discover, pull

- Setup a few environment variables.  
  ```bash
  export PORT=5000
  export REGISTRY=localhost:${PORT}
  export REPO=net-monitor
  export IMAGE=${REGISTRY}/${REPO}:v1
  ```
- Install the [ORAS client][oras-releases]
- Run a local instance of the [CNCF Distribution Registry][cncf-distribution]
  ```bash
  docker run -d -p ${PORT}:5000 ghcr.io/oras-project/registry:v0.0.3-alpha
  ```
- Build and Push `$IMAGE`
  ```bash
  docker build -t $IMAGE https://github.com/wabbit-networks/net-monitor.git#main
  docker push $IMAGE
  ```
- Push an SBoM
  ```bash
  echo '{"version": "0.0.0.0", "artifact": "'${IMAGE}'", "contents": "good"}' > sbom.json
  oras push $REGISTRY/$REPO \
      --artifact-type sbom/example \
      --subject $IMAGE \
      sbom.json:application/json

  echo '{"version": "0.0.0.0", "artifact": "'${IMAGE}'", "signature": "signed"}' > signature.json
  oras push $REGISTRY/$REPO \
      --artifact-type signature/example \
      --subject $IMAGE \
      signature.json:application/json
  ```
- List the tags, notice the additional metadata doesn't pollute the tag listing
  ```http
  curl $REGISTRY/v2/$REPO/tags/list | jq
  ```
- Get referenced artifacts with the `/referrers/` API
  ```bash
  DIGEST=$(oras discover $IMAGE -o json | jq -r .digest)
  curl $REGISTRY/oras/artifacts/v1/net-monitor/manifests/$DIGEST/referrers | jq
  ```
- Get a filtered list by `artifactType`
  ```bash
  curl "$REGISTRY/oras/artifacts/v1/net-monitor/manifests/$DIGEST/referrers?artifactType=sbom%2Fexample" | jq
  ```
- Get a filtered list with `oras discover`
  ```bash
  oras discover -o tree --artifact-type=sbom/example $IMAGE
  ```
- Pull a reference artifact by embedding `oras discover`
  ```shell
  oras pull -a \
      ${REGISTRY}/${REPO}@$( \
        oras discover  \
          -o json \
          --artifact-type sbom/example \
          $IMAGE | jq -r ".references[0].digest")
  ```

## Further Reading

- [Scenarios](./scenarios.md)
- [oras.artifact.manifest][artifact-manifest-spec]  spec for persisting artifacts
- [`/referrers/` API spec][referrers-api]  for discovering artifacts


[artifact-manifest-spec]:             ./artifact-manifest.md
[cncf-distribution]:                  https://github.com/oras-project/distribution
[oras-releases]:                      https://github.com/oras-project/oras/releases
[referrers-api]:                      ../manifest-referrers-api.md
