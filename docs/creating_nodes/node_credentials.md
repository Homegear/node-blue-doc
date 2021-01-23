# Node credentials

A node may define a number of properties as `credentials`. These are properties that are stored encrypted separately to the main flow file and do not get included when flows are exported from the editor.

To add credentials to a node, the following steps are taken:

First add a new `credentials` entry to the node’s definition:

```json
 credentials: {
    username: {type:"text"},
    password: {type:"password"}
 },
```

The entries take a single option - their `type` which can be either `text` or `password`.

Add suitable entries to the edit template for the node:

```html
 <div class="form-row">
     <label for="node-input-username"><i class="fa fa-tag"></i> Username</label>
     <input type="text" id="node-input-username">
 </div>
 <div class="form-row">
     <label for="node-input-password"><i class="fa fa-tag"></i> Password</label>
     <input type="password" id="node-input-password">
 </div>
```

Note that the template uses the same element `id` conventions as regular node properties.

For JavaScript nodes the call to `RED.nodes.registerType` must be updated to include the credentials, in the node’s `.js` file:

```javascript
 RED.nodes.registerType("my-node",MyNode,{
     credentials: {
         username: {type:"text"},
         password: {type:"password"}
     }
 });
```

## Accessing credentials

Todo