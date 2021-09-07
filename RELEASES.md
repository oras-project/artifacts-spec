# Specification Release Process

This process governs the motion of a specification from draft through full standardization. A specification is termed a _deliverable_ as it passes through various phases from _draft deliverable_ to _accepted deliverable_, at which point it becomes a final specification.

## Attribution

The specification release process was created using content and verbiage from the following specifications:

   * [Cloud Native Application Bundles](https://github.com/cnabio/cnab-spec/blob/main/901-process.md)
   * [OCI Distribution Specification](https://github.com/opencontainers/distribution-spec/blob/main/RELEASES.md)


## Versioning

A specification will call out a target version (such as 1.0.0), and then also include its stability marker (such as Draft, see below) to indicate progress towards the target version.

For example, the ORAS Artifacts specification should be formally referenced as _ORAS Artifacts 1.0.0_. The phase of the process MAY be appended as a stability marker: ORAS Artifacts 1.0.0-AD_.

Inspired by SemVer 2, Artifacts follows a rigid versioning scheme. Versions are presented in the form `X.Y.Z[-Sn]`, where `X` is the major version, `Y` is the minor version, and `Z` is the patch version. The optional `-Sn` is a draft stability marker and optional number index (1-n).

- Major releases (`X`): Major releases contain breaking changes, including features, fixes, and reorganizations. Implementors should not assume that two major versions are compatible. For example, `1.9.9` is not to be considered compatible with `2.0.0`.
- Minor releases (`Y`): Minor releases contain features and fixes only. A feature MUST NOT remove or modify existing pieces of the spec (including schemata and file system layouts), but MAY add new things. A minor release SHOULD be backward compatible, though certain security concerns may override this requirement.
- Patch releases (`Z`): Patch release contain fixes to the text of the specification. Patch releases MUST NOT change the behavior of the specification (except in cases where the specification was too vague and the patch clarifies).
    - Patch releases MUST be both forward and backward compatible to the minor version number
    - Patch releases MUST NOT make the schema harder to validate against (though they may relax the schema).

Stability markers provide a way to indicate maturity towards a target release and may incorporate features or fixes. If an object is tagged with a stability marker, it MUST be treated as incompatible with any other version number. E.g. `1.0.0-RC` MUST be considered incompatible with `1.0.0`. Production artifacts SHOULD NOT use stability markers.

A small number of stability markers are allowed as defined below:

- `DRAFT`: Draft indicates that the version is an unstable in-development version. `DRAFT` releases have no compatibility guarantees between versions and are used to quickly iterate.
- `RC`: Release Candidate indicates that the version has completed planned development is used to gather feedback from the community prior requesting final approval from the ORAS organization maintainers.
- `AD`: Final approval by the ORAS organization maintainers.

Both the `DRAFT` and the `RC` may include an optional number suffix to indicate that there may be multiple releases within a specific stability marker. eg `oras-artifacts-spec-1.0.0-DRAFT1. This enables implementors to verify and provide feedback against a specific release whilst in earlier stability markers.

The tag `AD` should never be used in a SemVer stability marker. `AD` is synonymous with the final release.

Content persisted MUST be version specific via the use of the `"mediaType"` field to assure the specification version is not assumed. For example code implementing `oras-artifacts-spec-1.0.0.DRAFT1` must use `"mediaType": "application/vnd.cncf.oras.artifact.manifest.1.0.0-DRAFT1+json"`.

The stability markers `ALPHA`, `BETA`, and so on are _disallowed_ under this specification, and MUST NOT be used to express Artifacts specification versions.

Finally, certain small errata may be fixed on an existing release without incrementing the release version. The following changes are allowed as errata fixes:

- Correcting spelling or typographical errors, where changing these does not alter the meaning of the specification.
- Correcting minor grammatical mistakes.
- Adding a revised link when a broken link appears. This should be done by appending the text `(Updated link: http://example.com...)`. The text may be corrected fully during the next version change.
- In extenuating circumstances, the ORAS organization maintainers may approve retroactively editing text to meet legal requirements. In such cases, the ORAS organization maintainers will not approve changes that break the specification. Under such circumstances, the ORAS organization maintainers may issue a _retraction of a specification_ (removing a published specification), and publish a new specification version that meets the legal requirements. For example, an intellectual property infringement may only be correctable by a retraction.
Additional properties that standardize communications in transit. 

## Git Release Flow

This section deals with the practical considerations of versioning in Git, this repo's version control system.

### Patch releases

When a patch release of a specification is required, the subproject maintainers must approve the scope of commits proposed for inclusion. The patch commit(s) should be merged to the `main` branch when ready. Next, a new branch should be created for the designated patch. For example, if the previous most recent branch name of the specification is `oras-artifacts-spec-1.0.0`, the new branch would be created from `oras-artifacts-spec-1.0.0` and named `oras-artifacts-spec-1.0.1-rc`. The patch commit(s) should then be cherry-picked into this new branch.

When the final release is approved, a Git tag should also be pushed, which triggers schema artifact publishing. Extending the example above, a `oras-artifacts-spec-1.0.1-ad` tag should be created from the `oras-artifacts-spec-1.0.1` branch and pushed to origin. We drop the `-ad` suffix as branches and tags may not have the same name in Git.

### Minor releases

When a minor release of a specification is required, the subproject maintainers must approve the scope of commits proposed for inclusion. Likely this will be the `main` branch once the approved commit(s) are merged. Next, a new branch should be created from `main` and named `oras-artifacts-spec-1.1.0-rc` (here assuming that the version immediately prior was `oras-artifacts-spec-1.0.0`).

When the final release is approved, a Git tag should also be pushed, which triggers schema artifact publishing. Extending the example above, a `oras-artifacts-spec-1.1.0-ad` tag should be created from the `oras-artifacts-spec-1.1.0` branch and pushed to origin. We drop the `-ad` suffix as branches and tags may not have the same name in Git.

### Major releases

When a major release of a specification is required, the subproject maintainers must approve the scope of commits proposed for inclusion. Likely this will be the `main` branch once the approved commit(s) are merged. Next, a new branch should be created from `main` and named `oras-artifacts-spec-2.0.0-rc` (here assuming that the version immediately prior was `oras-artifacts-spec-1.0.0`).

When the final release is approved, a Git tag should also be pushed, which triggers schema artifact publishing. Extending the example above, a `oras-artifacts-spec-2.0.0-ad` tag should be created from the `oras-artifacts-spec-2.0.0` branch and pushed to origin. We drop the `-ad` suffix as branches and tags may not have the same name in Git.

### Ad hoc schema releases

In addition to the scenarios above, if schemas at a certain commit need to be preserved in the form of artifacts and published, ad hoc versioning (i.e. not tied to a release branch) is permitted via the following Git tag flow. This is intended for specifications in `DRAFT` state which are perhaps under heavy development.

To enable implementations to pin at a certain version prior to an official release, we can issue a Git tag and CI will handle publishing the schemas. For example, if the specification is still in the `DRAFT` state but its schemas at a checkpoint are needed for implementation verification, we can push an appropriate tag to origin. The tag form is: `oras-artifacts-spec-1.0.0-DRAFT+abc1234`, where `oras-artifacts-spec-1.0.0-DRAFT` is the current working version and `abc1234` is the short SHA of the commit the tag will be created from.

## Development Process

The specification will proceed through the following phases:

- *Draft (Draft):* A Pre-Draft may be approved by the subproject, in which case it becomes an official draft under the auspices of the ORAS Artifacts spec subproject. The subproject will continue to revise the draft until it is in a state the group sees as fit for standardization.
- *Release Candidate (RC):* When the subproject believes feature work has been completed and is awaiting feedback from implementors. 
- *Final Approval (AD):* The ORAS organization maintainers may grant Final Approval to a draft with subproject approval. At this point, the work is now designated an Approved Deliverable and is no longer a draft.

Documents that have reached AD are considered complete. Errata may be captured in a separate section of the document, but the document itself is not changed except to correct typographical and formatting errors where necessary.

When the content of a document needs changes that cannot be captured as errata, a new _version_ of the specification must be created, and must proceed through the stages outlined above.
