# Synapse-Workflow-Examples User Guide

Synapse-Workflow-Examples provides examples and documentation to assist
in building Optic Workflows.

To try out the example Workflows, ensure the Synapse-Workflow-Examples
Power-Up is installed and navigate to the Workflows tool in Optic. From
there, expand the `synapse-workflow-examples` package to view the
Workflows.

To view the YAML for each example, use the `Workflow Example Code`
Workflow. It contains a `datatable` Element listing the examples and
will display the example Workflow YAML in a `textarea` Element.

It is highly recommended to validate Workflow YAML with the provided
[JSON Schemas](#json-schemas) in both development (in editor) and CI/CD
test suites.

## Optic Workflows

Workflows can be used to build dynamic UX from provided Elements and
Storm, and are defined using a declarative API that is native to Optic.
The defined UX is then integrated directly into Optic via Storm packages
with one or more Workflows being defined in the package definition.

A Workflow primarily consists of a set of Elements that display and
allow interaction with data in a variety of ways. The Elements defined
in a Workflow will specify a type that will dictate its display and
capabilities. Some Element types include a configurable form, two types
of tables, a node viewer, and a dynamic syntax highlighting textarea.
Most types of Elements can execute Actions in response to user input to
propagate and display dynamic data, allowing Elements to share data in a
reactive way. Other types of Actions include updating internal
variables, opening other Workflows, and running Storm queries to modify
and pull in additional data.

Elements typically execute Actions in response to Events or user input.
An Element can subscribe to Events from other Elements in the Workflow.
Once a subscription has been established, the Element can define a set
of Actions that will be executed in response to a particular Event from
the source Element. There are several types of Events, with more likely
to be added. Some Events carry data that can be accessed while executing
Actions (onvars and onnodes).

Elements additionally maintain a set of variables (vars) that can be
manipulated using Actions and passed to and from Storm queries. These
vars are kept in the Browser and are accessible by the Element for the
duration of the Workflow session. This allows for dynamic behavior based
on the vars that may come from user input, other Elements, Storm
queries, or any combination of the three. Elements can reference these
vars for get and set operations in their opts and Actions they execute.
They can be referenced simply by name `myvar`, or using a `$` for
clarity: `$myvar`. Nested dict and array values can be referenced using
dotted syntax e.g. `$mydict.somekey` and `$myarray.0`.

Events and Actions together a flexible framework to allow Elements to
react to user input and changes in other Elements, all the way through
to running Storm with user provided input.

Workflows can also be built to work together to allow opening other
Workflows in a modal, navigating to other Workflows, and allow opening a
Workflow in the Research tool via special Node actions.

When designing a Workflow one should consider the following:

-   Elements - What to display and how to display it.
-   Layout - How will the Elements be visually laid out in the grid.
-   Events - Which Events are suitable for distributing the necessary
    data between the selected Elements.
-   Actions - What types of Actions will executed by each element when
    receiving a particular Event or User input.
-   Coordination - What other Workflows may be opened as a modal or
    navigated to. What other Workflows may want to be able to open or
    navigate to this one.

## Workflow Events

A Workflow Event is a message that will be broadcast through the
Workflow, and sometimes into other Workflows. Elements can Subscribe to
Events from other Elements to receive them and then define their Actions
to be executed based on (new) data contained in the Event(s).

There are currently two types of Events each with a specific use.

### onvars

This Event can be fired by other Elements or External Workflows. Its
purpose is to propagate the source Elements vars to any other subscribed
Elements. Any Element that has a subscription to onvars Events from a
particular source Element will automatically get the inbound vars set on
itself. When reacting to an `onvars` Event, Elements will often rerender
themselves with new data, either directly from the Events vars or after
making additional Storm queries to gather more data or Nodes to be
rendered.

### onnodes

This Event can fired by other Elements to pass one or more Nodes between
Elements. When executing Actions in response to this Event, the nodes
will be available for both `updatevars` and `nodes` operations and in
Storm queries as inbound nodes via the `idens` query opt.

## Workflow Actions

An Action is an operation executed by an Element in response to either
an Event or user input. There are many types of Actions that can have a
wide variety of effects on the UX of the Workflow.

### eventfire

Fire an Event into the Workflow, allowing other subscribed Elements to
react to the provided data.

### nodesfire

Fire an `onnodes` Event into the Workflow. Typically using the
*selected* Nodes in an Element that supports Node selection e.g
`stormtable` using the `selected` opt or the Nodes in an `onnodes` Event
being handled nodes using `propagate`.

### openmodal

Open another Workflow in a modal. There are additional opts to provide
data to the target Workflow, and to make temporary subscriptions to its
Events so that the source Element can update as desired.

### closemodal

Close a specific modal. Mostly useful in the Actions defined in a
temporary subscription so that the source Workflow has control over when
the modal will be closed.

### navworkflow

Navigate to directly to another Workflow. Can additionally send an
`onvars` or `onnodes` Event into the Workflow after it\'s been rendered,
to provide data for initial render if the target Workflow supports it.

### researchquery

Provide a query or a var to get a query from, and navigate the user to
the Research tool to execute it.

### storm

Run a Storm query and render the yielded Nodes according to the Element
type. The executing Element\'s vars will be passed in for the query to
use. This Action has a variety of opts that can change its behavior.

One of either `query` or `queryvar` must be set to provide the query to
execute. All other opts are optional.

**query**

> The Storm query that should be executed. Nodes yielded from the query
> will be consumed by the executing Element to be rendered according to
> the Element type.

**queryvar**

> Resolve the Storm query to execute from a var on the Element.

**inboundnodes**

> Allows supplying inbound nodes to the query from a var when not
> handling an `onnodes` Event. This is useful when it is necessary to
> inbound nodes to a query that is not in direct reaction to an
> `onnodes` event. Nodes are inbounded to the query in the same way as
> handling an `onnodes` Event via the Storm query opt `idens`.
>
> Note that this opt will be *ignored* when the Action is handling an
> `onnodes` Event.
>
> Example:
>
>     # handle an onnodes event and stash the nodes in the $stashednodes var.
>     events:
>       onnodes:
>         - type: updatevars
>           opts:
>             operations:
>               - type: nodes
>                 name: $stashednodes
>                 path: $nodes
>
>     ... the rest of your Workflow ...
>
>     # later (e.g. user button click or other interaction) run a Storm query with the nodes from $stashednodes inbound.
>     type: storm
>     opts:
>       # pivot from the inbound nodes to inet:ipv4 nodes
>       query: -> inet:ipv4
>       inboundnodes: $stashednodes

**render**

> Can be set to `false` to cause a `storm` Action rendering Element to
> *not* render the results of the Storm query.

**spinner**

> Configure any spinners that should reflect the running status of the
> query.
>
> Can be set to `true` to bind to the spinner on the executing Element.
>
> Can be set to the iden of another Element to bind to the spinner on
> that Element.
>
> Can be set to a dict with `element` and `button` values to spin a
> button on a particular Element (the Element can be itself as well).
>
> `button` requires `idx` which is the index of the button in the
> Element. Element titlebar buttons are first, followed by buttons
> created by the Element type. Can also provide temporary `spinningtext`
> to update the text of the button while the spinner is active.
>
> Example:
>
>     spinner:
>       element: <elementiden>,
>       button:
>         idx: 0
>         spinningtext: 'Updating...'

**feed**

> Configure other Elements to feed messages coming out of the Storm
> query. Messages fed to other Elements can optionally be limited by
> type. This allows other Elements to be in control of the `storm`
> Action, while still feeding relevant messages to other Elements in the
> Workflow. e.g. a `querybar` executing a query that yields nodes would
> want to feed those nodes into a `stormtable` or some other Element,
> while additionally feeding messages to other suitable elements.
>
> Example:
>
>     feed:
>       ex-stormtable: [node]
>       ex-other-selective: [node, warn, err]
>       ex-other-all: true

**console**

> Optionally configure Storm messages to be output to either a
> `stormconsole` Element in the Workflow or the \'global\' Console tool
> Console.
>
> Workflow Element Example:
>
>     console:
>       iden: ex-stormconsole
>
>       clear: true
>
>       # optionally output only specified message types
>       # mesgs: [print, warn, err]
>
> Global Console Example:
>
>     console:
>       global: true
>
>       # optionally output only specified message types
>       # mesgs: [print, warn, err]

**queryopts**

> Extra query opts to be used when executing the Storm query. These opts
> are documented in the [Synapse Storm
> Opts](https://synapse.docs.vertex.link/en/latest/synapse/devguides/storm_api.html#storm-opts)
> documentation. Note: `view`, `idens`, and `vars` opts are not allowed.

### callstorm

Run a query using callStorm and handle the return value according to the
Element type. The executing Element\'s vars will be passed in for the
query to use.

**query**

> Required Storm query that will be executed. The query must
> `return(<something>)`. The value the query should return is dependent
> on the executing Element.

**inboundnodes**

> Allows supplying inbound nodes to the query from a var when not
> handling an `onnodes` Event. This is useful when it is necessary to
> inbound nodes to a query that is not in direct reaction to an
> `onnodes` event. Nodes are inbounded to the query in the same way as
> handling an `onnodes` Event via the Storm query opt `idens`.
>
> Note that this opt will be *ignored* when the Action is handling an
> `onnodes` Event.
>
> Example:
>
>     # handle an onnodes event and stash the nodes in the $stashednodes var.
>     events:
>       onnodes:
>         - type: updatevars
>           opts:
>             operations:
>               - type: nodes
>                 name: $stashednodes
>                 path: $nodes
>
>     ... the rest of your Workflow ...
>
>     # later (e.g. user button click or other interaction) run a callStorm query with the nodes from $stashednodes inbound.
>     type: callstorm
>     opts:
>       inboundnodes: $stashednodes
>
>       # do some operation on the nodes
>       query: |
>         [ :someprop=$newvalue ]
>         return($lib.true)

### loadnodes

Render the Nodes present in an `onnodes` Event payload according to the
Element type.

### selectnodes

Select Nodes that have already been rendered in the Element. Note this
is only useful in Elements that support Node selection, e.g. tables.

### getvars

Run a query using callStorm that returns a \$lib.dict of vars that will
be set onto the Element.

### updatevars

Update vars on the Element with a series of operations of different
types. Each operation has `type` to specify the type of operation and
`name` to specify the var to update.

**set**

> Set a var to a static value.
>
> Example:
>
>     type: updatevars
>     opts:
>       operations:
>         - type: set
>           name: $myvar
>           value: 'My Static Value'

**var**

> Set a var based on another var. Useful for plucking out nested vars to
> the top level as needed.
>
> Example:
>
>     type: updatevars
>     opts:
>       operations:
>         - type: var
>           name: $myvar
>           var: $somenesteddata.subkey.foo

**fmtstr**

> Set a var to a string using a format string syntax. Existing vars can
> be referenced, along with nodes in a `onnodes` Event currently being
> handled.
>
> Var Example:
>
>     type: updatevars
>     opts:
>       operations:
>         # using the $somevar var
>         - type: fmtstr
>           name: $mystr
>           fmtstr: 'Hello {$somevar}!'
>
> Nodes Example:
>
>     # handling on onnodes Event
>     events:
>       onnodes:
>         - type: updatevars
>           opts:
>             operations:
>               # using the first node from the onnodes Event we are currently handling
>               - type: fmtstr
>                 name: $mystr
>                 fmtstr: 'Hello {$nodes.0.props.name}!'

**toggle**

> Toggle the boolean value of a var. If the var is not set, the first
> toggle will set the value to `true`. If the var exists and is a
> boolean, it will be toggled. If the var exists and is not boolean a
> warning will be logged.
>
> Example:
>
>     type: updatevars
>     opts:
>       operations:
>         - type: toggle
>           name: $mytoggle

**del**

> Delete a var, resetting it to be undefined.
>
> Example:
>
>     type: updatevars
>     opts:
>       operations:
>         - type: del
>           name: $bye

**callstorm**

> Set a var to the return value of a Storm query executed with
> callStorm.
>
> Example:
>
>     type: updatevars
>     opts:
>       operations:
>         - type: callstorm
>           name: $mystormreturn
>           query: return(([1, 2, 3]))

**nodes**

> Set a var from the nodes in the `onnodes` Event currently being
> handled.
>
> Example:
>
>     # handling on onnodes Event
>     events:
>       onnodes:
>         - type: updatevars
>           opts:
>             operations:
>               # store the :name prop of the first node
>               - type: nodes
>                 name: $nodename
>                 path: $nodes.0.props.name
>
>               # store the entire node(s) for use with storm / callstorm Action inboundnodes opt.
>               - type: nodes
>                 name: $stashednodes
>                 path: $nodes

### updateopts

Update opts on the Element with a series of operations of different
types.

### refresh

Refresh the Element from potentially updated opts and vars.

### hidenodes

Hide all Nodes or the Nodes in the `onnodes` Event if reacting to an
`onnodes` event.

### toast

Display a toast popup message to let the user know about success,
warnings or errors.

### stop

Stop the current chain of Actions. Typically used with a filter to bail
out of an Action chain after a warn/error toast.

### loadingmodal

Display or hide a modal loading spinner during the wait for other
operations to complete.

### sendfocus

Send focus to a field in a `stormform`, `querybar`, `queryeditor`, or
`tabs` Element. For a `stormform` the `field` opt is required and for
`tabs` Element the `tabname` opt is required.

## Elements

Elements are the primary unit of composition in a Workflow. Each Element
can have one or more responsibilities in the Workflow, including
displaying data/nodes, accepting user input, running Storm queries that
CRUD nodes. Every Element has a set of common top-level properties:

-   type - The Element Type for the Element.
-   title - The title for the Element.
-   doc - Can provide a tooltip and/or markdown content to be displayed
    with the Element.
-   layout - The position and dimensions of the Element in the grid.
-   disabled - If the Element should be disabled. This can be a var to
    make it dynamic when the var is updated, e.g.
    `disabled: $foodisabled`
-   hidden - If the Element should be hidden. This can be a var to make
    it dynamic when the var is updated, e.g. `hidden: $foohidden`
-   oninit - A set of Actions to execute *before* the Element is
    rendered.
-   onload - A set of Actions to execute immediately after creation of
    the Element.
-   subs - Defines subscriptions to other Elements Events.
-   events - Defines Actions to be taken when receiving Events.

### Element Lifecycle

An Element can optionally define `oninit` and/or `onload` to execute
Actions at specific points in its lifecycle. The `oninit` hook allows
executing Actions *before* the Element is rendered, and the `onload`
hook allows executing Actions just *after* all Elements in the Workflow
have been rendered.

### Element Types

#### Buttons: `buttons`

A simple element that can display buttons that define Actions to be
executed when they are clicked.

The state of the buttons can be dynamically updated using the
`updateopts` Action, followed by a `refresh` Action.

#### TextArea: `textarea`

An Element that provides a textarea powered by CodeMirror.

It can be populated statically, from a var, from a Node prop, or from a
callStorm query.

There are additionally options to control display, including syntax
highlighting modes and line wrapping.

Actions to be executed when the TextArea value is edited can be defined
in `onchange`.

#### DataTable: `datatable`

An Element that can can display arbitrary data in a `record` format as a
list of Storm `$lib.dict()` objects. This is achieved using the
[\$lib.fire()
API](https://synapse.docs.vertex.link/en/latest/synapse/autodocs/stormtypes_libs.html#lib-fire-name-info)
and a type of `optic:datatable:row`.

Columns are defined in the `columns` opt, and should specify a `key`
that will grab the appropriate property from each record for the column.

The Element can additionally configure its `opts` dynamically using
either an `updateopts` Action, or when streaming messages in a `storm`
Action if appropriate. If a `storm` Action fires a
`optic:datatable:init` message then it can contain an `opts` dictionary
that will be used to update `opts` and refresh, before continuing to
process `optic:datatable:row` messages.

See the `Datatable` and `Datatable - Dynamic` Workflows for example
usage.

#### NodeEditor: `nodeeditor`

An Element that can present fields for creating/editing Nodes.

Fields can be automatically generated based on a node `form`, or custom
defined fields.

The values are gathered into the `$fields` var and will be handed into
the provided query.

#### NodeViewer: `nodeviewer`

An Element that can display primary values, props, embeds, tags and
tagprops of a particular Node.

#### StormForm: `stormform`

An Element that can display a defined set of fields in a form. Fields
can be of different types, including `input`, `dropdown` and
`synforminput`.

Each field is mapped to a var by `name` and will be updated anytime the
field\'s value changes. Actions can optionally be executed whenever a
field\'s value is changed using `onchange`, which allows propagating
vars to other Elements, and causing an internal refresh to update other
fields in the form Element. All fields are additionally bundled into a
`$fields` dict containing populated values by each field\'s name for
convenience.

A set of buttons can also be provided using the `buttons` opt. Buttons
are defined in the same way as the `buttons` Element.

In addition to the normal `callstorm` behavior where the query can
return a dict that will be used to update the Elements vars. It is also
possible to provide a `fielderrs` key in the returned dict. If provided,
the Element will expect the value to be a dict of field `name` to an
error message to be displayed just below the field to provide user
feedback in validation / error situations.

#### StormTable: `stormtable`

An Element that displays Nodes of a particular form. The table behaves
similarly to the Tabular display mode. Columns can be defined, to
replicate any configuration achievable in the Tabular display mode.
Actions can additionally be defined for `onselect` and menu options, to
execute Actions with the target Node(s).

#### QueryBar: `querybar`

A single line \<input\> based query bar with a query running spinner.
\<Enter\> will cause the Actions defined in `onrun` to be executed.

#### QueryEditor: `queryeditor`

A multiline syntax highlighting query editor with a query running
spinner. \<Shift+Enter\> will cause the Actions defined in `onrun` to be
executed.

#### Tabs: `tabs`

A tabs layout Element that allows defining tabs with Elements to be
displayed in them when the tab is selected. The tab Elements will be
displayed in their own Grid so all layout is relative to the parent.

### JSON Schemas

JSON Schemas that document and validate Workflows and Packages
containing Workflows are available to download from your Optic instance:

> Schema for a standalone Workflow yaml file
>
> :   `https://<youroptic>/schemas/workflow.schema.json`
>
> Schema for a package yaml file
>
> :   `https://<youroptic>/schemas/pkg-workflows.schema.json`

### Examples

Workflows are defined under the top-level `optic` key in a Storm package
definition. Individual workflow definitions can be included in-line, or
as a separate file in `workflows/<workiden>.yaml`. Files from the
`workflows/` directory are automatically embedded when the Synapse
genpkg tool is used.

Workflow and Element `iden` values (keys in definition) must be unique,
and can only include alphanumeric, `_`, or `-` characters. To avoid URL
path collisions, workflow idens should be namespaced, e.g.
`<pkgname>-<workflow name>`.

``` yaml
name: foopkg
version: 1.0.0

modules:
  - ...

commands:
  - ...

optic:
  displays:
    - title: Workflow title for Optic sidebar
      workiden: foopkg-inline
    - title: Another title
      workiden: foopkg-fromdir

  workflows:
    foopkg-inline:
        desc: An inline complete workflow definition
        ...

    # Additional workflows from ``workflows/`` directory would be included here by the Synapse genpkg tool
```

Packages can be generated (and pushed) using the [Synapse genpkg
tool](https://synapse.docs.vertex.link/en/latest/synapse/userguides/syn_tools_genpkg.html).
