# Synapse-Workflow-Examples Admin Guide

## Configuration

Optic Workflows is in beta and is controlled by a feature flag in your
Optic instance, and is enabled by default. See the documentation for
Optic feature flags
[here](https://synapse.docs.vertex.link/projects/optic/en/latest/user_interface/devopsguide.html#enable-alpha-beta-features)
for more information on controlling Optic features.

Synapse-Workflow-Examples requires certain data to be available in the
current view, and in some cases permissions, in order to fully
demonstrate the example.

### Data Used by Examples

The `Project Management` workflow requires at least one `proj:project`
with at least one associated `proj:ticket`.

The `Form + Table + Viewer` workflow requires at least one `ou:org` node
that has the `:name` property populated. The `contacts` and `hosts`
tables will be populated by performing pivots from `ou:org` to
`ps:contact` and `it:host` respectively.

### Permissions

In order to use the `Node Creator` workflow the user must have
permissions to add nodes in the current view.

## Exported APIs

Synapse-Workflow-Examples does not currently export any APIs.

## Node Actions

Synapse-Workflow-Examples does not currently provide any Optic Node
Actions.

## Onload Events

Synapse-Workflow-Examples does not use any `onload` events.
