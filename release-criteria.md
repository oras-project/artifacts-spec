# Release Criteria

The artifacts-spec represents content stored in a registry. Once available to users to persist their artifacts, they will have expectations for that content to be available indefinitely.

This document captures the target releases and scope of the releases to enable triaging of issues and PRS for resolution or deferment.

## 1.0-draft[n] - Initial releases for validations

The `1.0-draft[n]` releases will be used for end to end (e2e) validations, prior to a level of confidence content may be indefinitely persisted. 

The `-draft[n]` releases are intended to lock-down potential future breaking changes, close on schemas, APIs and parameter names.

Content persisted with a `-draft[n]` version has no expectation it may be supported in a future release.

Content that is persisted (pushed) with `-draft[n]` MAY fail PULL as the registry completes the release. Users must always be able to delete persisted content. However, depending on the specific capability, this may require deleting the repo within the registry.

Content persisted with these versions MUST use the `"mediaType": application/vnd.cncf.oras.artifact.manifest.1.0-draft[n].json` version to assure it's not assumed to be compliant with committed, released builds.

### 1.0-draft[n] - Acceptable issues and PRs

1. Additional properties
1. Removing Properties
1. Property Renames
1. Breaking changes to persisted content
1. Additional clarification that may loosen definitions of what may be persisted
1. Additional clarification that may restrict content previously enabled
1. Samples and docs that clarify use cases
1. Issues related to content copying within and across registry implementations

### 1.0-draft[n] - Non-acceptable PRs

1. Fundamental redesigns that change the use of a new manifest
1. Fundamental redesigns that change the use of a `referrers/` api
1. Changes that would make it impossible to delete the content from the registry in a released version.
   Exceptions include the ability to delete a repository within a registry.
   The user shouldn't have to delete all content in the registry to purge a `-draft[n]` artifact-manifest.

## rc[n] - Release Candidates

A target of 2-3 `-RC[n]` releases will be made, enabling stabilization with additional, non-breaking changes to persisted content and APIs for storing, discovering or retrieving content.

### rc[n] - Acceptable Issues and PRs
1. Additional properties
1. Additional clarification that may loosen definitions of what may be persisted
1. Samples and docs that clarify use cases

### rc[n] - Non-acceptable PRs
1. API, Parameter, Existing Schema Renames
2. Changes that may cause failures in retrieving artifact-manifest content, including the traversal of references.

## 1.n Releases

1.n releases represent minor additive changes to the 1.0 spec. By providing 1.n versioning, the client and registry clearly know there may be new content or capabilities included. By providing 1.n versioning, the artifacts-spec enables reliable expectations between the various registry clients and the registry services.

### 1.n - Acceptable Issues and PRs

1. Additional properties that standardize communications in transit. 
2. Clarifications on properties, schemas and apis that fill gaps in the expectations of usage. Clarifications SHOULD NOT block scenarios that are declared obvious usages of a previous minor release.

### 0.n - Non-acceptable PRs

1. Additional properties that may leak expectations a server may be processing the property.

## 2.n Releases

Major functional changes, where the registry needs to declare it's support of a group of new capabilities.

### 2.n Acceptable Issues and PRs

1. Major additive capabilities
1. Expectations of server processing to enable a capability. For instance, associating updateable meta-data to a subject artifact

### 2.n Non-acceptable Issues and PRs

1. Changes in the APIs that would disable the persistance, discovery, retrieval of previous versions.
2. APIs that have different behaviors, such as changing the result from a flat list to a hierarchal result would require versioning of the API. example: `/oras/v2/referrers`