# Node credentials

A node may define a number of properties as `credentials`. These are properties that are stored encrypted separately to the main flow file and do not get included when flows are exported from the editor.

## Adding credentials

To add credentials to a node, the following steps are taken:

### Node definition

First add a new `credentials` entry to the node’s definition:

```json
 credentials: {
    username: {type:"text"},
    password: {type:"password"}
 },
```

The entries take a single option - their `type` which can be either `text` or `password`.

### Node template

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

### Credential type file

The runtime needs to know the credential types. For this a file named `<type>.cred.json` must be created in the same directory as the code file. I. e. if your code file is named `my-node.php`, the credential type file must be named `my-node.cred.json`:

```javascript
 {
     "username": "text",
     "password": "password"
 }

```

## Accessing credentials

### Runtime use of credentials

Todo

### Credentials within the Editor

Within the editor, a node has restricted access to its credentials. Any that are of type `text` are available under the `credentials` property - just as they are in the runtime. But credentials of type `password` are not available. Instead, a corresponding boolean property called `has_<property-name>` is present to indicate whether the credential has a non-blank value assigned to it.

```javascript
oneditprepare: function() {
    // this.credentials.username is set to the appropriate value
    // this.credentials.password is not set
    // this.credentials.has_password indicates if the property is present in the runtime
    ...
}
```