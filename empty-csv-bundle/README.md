## prerequisites
- `opm` installed from https://github.com/operator-framework/operator-registry
- podman (docker is supported as well but I only tested with podman)
- image repositories for
  - the bundle image
  - the index image

## steps:
### create bundle image
The bundle image contains the manifest files for the actual deployment of a __single__ version of an operator.
The format is described here: https://github.com/operator-framework/operator-registry#manifest-format

Running `podman build -t quay.io/erdii/empty-csv-bundle:0.0.1 .` in the current folder will build the bundle, consisting of:
- the manifests folder which, at-minimum, MUST include a ClusterServiceVersion
- metadata/annotations.yaml which SHOULD conform to the same labels that are specified in the Dockerfile

Since the `opm` tool expects bundle images to be pullable, you have to push it to a registry:
`podman push quay.io/erdii/empty-csv-bundle:0.0.1`

### create the index image
The process is described here: https://github.com/operator-framework/operator-registry#building-an-index-of-operators-using-opm

To build an index image for the bundle run: `opm index add --build-tool podman --bundles quay.io/erdii/empty-csv-bundle:0.0.1 --tag quay.io/erdii/empty-csv-index:0.0.1`
Push the index image to quay: `podman push quay.io/erdii/empty-csv-index:0.0.1`

You are can now reference this index image in a `CatalogSource` object: `kubectl apply -f catalogsource.yaml`
This should eventually result in an available `PackageManifest`: `k get packagemanifest empty-csv`

### stuff
Further reading on opm: https://github.com/operator-framework/operator-registry/blob/master/docs/design/opm-tooling.md
