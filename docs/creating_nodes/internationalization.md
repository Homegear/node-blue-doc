# Internationalization

If a node is [packaged](packaging.md) as a proper module, it must include a message catalog in order to provide translated content in the editor.

For each node identified in the module’s `package.json`, a corresponding set of message catalogs and help files can be included alongside the node’s code file.

Given a node identified as:

```json
"name": "my-node-module",
"node-red": {
    "myNode": "myNode/my-node.js"
}
```

The following message catalogs may exist:

```
myNode/locales/__language__/my-node.json
myNode/locales/__language__/my-node.help.html
```

The `__language__` part of the path identifies the language the corresponding files provide. By default, Node-RED uses `en-US`. For every supported language a 2 letter language code should be added as well (this can be done as soft link). For `en-US` this would be `en`.

### Message catalog

The message catalog is a JSON file containing any pieces of text that the node may display in the editor.

For example:

```json
{
    "myNode" : {
        "message1": "This is my first message",
        "message2": "This is my second message"
    }
}
```

The catalog is loaded under a namespace specific to the node. For the node defined above, this catalog would be available under the `my-node-module/myNode` namespace.

The core nodes use the `node-red` namespace.

### Help text

The help file provides translated versions of the node’s help text that gets displayed within the Info sidebar tab of the editor.

### Palette help text

Palette help text is placed within the catalog JSON file using the key `paletteHelp`:

```
{
    "myNode" : {
        "message1": "This is my first message",
        "message2": "This is my second message",
        "paletteHelp": "This node is a dummy node to explain where help texts are placed"
    }
}
```



## Using i18n messages

In the editor functions are provided for nodes to look-up messages from the catalogs. These are pre-scoped to the nodes own namespace so it isn’t necessary to include the namespace in the message identifier.

### Editor

### HTML

Any HTML element provided in the node template can specify a `data-i18n` attribute to provide the message identify to use. For example:

```html
<span data-i18n="myNode.label.foo"></span>
```

By default, the text content of an element is replace by the message identified. It is also possible to set attributes of the element, such as the `placeholder` of an `<input>`:

```html
<input type="text" data-i18n="[placeholder]myNode.placeholder.foo">
```

It is possible to combine these to specify multiple substitutions For example, to set both the title attribute and the displayed text:

```html
<a href="#" data-i18n="[title]myNode.label.linkTitle;myNode.label.linkText"></a>
```

### JavaScript

All node definition functions (for example, `oneditprepare`) can use `this._()` to retrieve messages:

```javascript
this._("myNode.label.foo");
```

