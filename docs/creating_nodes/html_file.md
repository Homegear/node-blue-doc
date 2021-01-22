# HTML file

This section is largely citing the Node-RED documentation.

The node `.html` file defines how the node appears with the editor. It contains three distinct parts, each wrapped in its own `<script>` tag:

1. the main node definition that is registered with the editor. This defines things such as the palette category, the editable properties (`defaults`) and what icon to use. It is within a regular javascript script tag
2. the edit template that defines the content of the edit dialog for the node. It is defined in a script of type `text/html` with `data-template-name` set to the [type of the node](#node-type).
3. the help text that gets displayed in the Info sidebar tab. It is defined in a script of type `text/html` with `data-help-name` set to the [type of the node](#node-type).

## Defining a node

A node must be registered with the editor using the `RED.nodes.registerType` function.

This function takes two arguments; the type of the node and its definition:

```html
<script type="text/javascript">
    RED.nodes.registerType('node-type',{
        // node definition
    });
</script>

```

### Node type

The node type is used throughout the editor to identify the node. It must match the type in the corresponding code file.

### Node definition

The node definition contains all of the information about the node needed by the editor. It is an object with the following properties:

- `category`: (string) the palette category the node appears in
- `defaults`: (object) the [editable properties](node_properties) for the node.
- `credentials`: (object) the [credential properties](node_credentials) for the node.
- `inputs`: (number) how many inputs the node has, either `0` or `1`.
- `outputs`: (number) how many outputs the node has. Can be `0` or more.
- `color`: (string) the [background colour](node_appearance) to use.
- `paletteLabel`: (string|function) the [label](node_appearance#label) to use in the palette.
- `label`: (string|function) the [label](node_appearance#label) to use in the workspace.
- `labelStyle`: (string|function) the [style](node_appearance#label-style) to apply to the label.
- `inputLabels`: (string|function) optional [label](node_appearance#port-labels) to add on hover to the input port of a node.
- `outputLabels`: (string|function) optional [labels](node_appearance#port-labels) to add on hover to the output ports of a node.
- `icon`: (string) the [icon](node_appearance#icon) to use.
- `align`: (string) the [alignment](node_appearance#alignment) of the icon and label.
- `button`: (object) adds a [button](node_appearance#buttons) to the edge of the node.
- `oneditprepare`: (function) called when the edit dialog is being built. See [custom edit behaviour](node_properties#custom-edit-behaviour).
- `oneditsave`: (function) called when the edit dialog is okayed. See [custom edit behaviour](node_properties#custom-edit-behaviour).
- `oneditcancel`: (function) called when the edit dialog is canceled. See [custom edit behaviour](node_properties#custom-edit-behaviour).
- `oneditdelete`: (function) called when the delete button in a configuration node’s edit dialog is pressed. See [custom edit behaviour](node_properties#custom-edit-behaviour).
- `oneditresize`: (function) called when the edit dialog is resized. See [custom edit behaviour](node_properties#custom-edit-behaviour).
- `onpaletteadd`: (function) called when the node type is added to the palette.
- `onpaletteremove`: (function) called when the node type is removed from the palette.

## Edit dialog

The edit template for a node describes the content of its edit dialog.

```html
<script type="text/html" data-template-name="node-type">
    <div class="form-row">
        <label for="node-input-name"><i class="fa fa-tag"></i> Name</label>
        <input type="text" id="node-input-name" placeholder="Name">
    </div>
    <div class="form-tips"><b>Tip:</b> This is here to help.</div>
</script>
```

More information about the edit dialog is available [here](node_edit_dialog).

## Help text

When a node is selected, its help text is displayed in the info tab. This should provide a meaningful description of what the node does. It should identify what properties it sets on outgoing messages and what properties can be set on incoming messages.

The help text is provided with the localization JSON file. You can find more information [here](help_style_guide).