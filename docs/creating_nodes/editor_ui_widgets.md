# Editor UI widgets

This section is largely citing the Node-RED documentation.

A set of jQuery widgets are available that can be used within a node’s edit template.

- [`TypedInput`](#TypedInput) - a replacement for a regular `<input>` that allows the type of the value to be chosen, including options for string, number and boolean. Used extensively in the core Node-BLUE nodes.
- [`EditableList`](#EditableList) - an editable list where the elements can be complex forms in their own right. Used by the core `Switch` and `Change` nodes.
- [`SearchBox`](#SearchBox) - an enhanced `<input>` for common usage around Search UX.
- [`TreeList`](#TreeList) - a list to display tree-structured data.

## TypedInput

A replacement for a regular `<input>` that allows the type of the value to be chosen, including options for string, number and boolean types.

### Options

#### `default`

*Type: String*

If defined, sets the default type of the input if `typeField` is not set.

#### `types`

*Type: Array*

The value of the option is an array of string-identifiers for the predefined types and `TypeDefinition` objects for any custom types.

The predefined types are:

| Identifier     | Description                                        |
| -------------- | -------------------------------------------------- |
| `msg`          | a `msg.` property expression                       |
| `flow`         | a `flow.` property expression                      |
| `global`       | a `global.` property expression                    |
| `string`       | a String                                           |
| `re`           | a Regular Expression                               |
| `int`          | a Integer                                          |
| `float`        | a floating point number                            |
| `suntime`      | time based on sun position (sunrise, sundown, ...) |
| `date`         | the current timestamp                              |
| `time`         | a `HH:MM[:SS]` input                               |
| `array`        | a JSON Array with JSON editor                      |
| `arraySimple`  | a JSON Array without JSON editor                   |
| `struct`       | a JSON Object with JSON editor                     |
| `structSimple` | a JSON Object without JSON editor                  |
| `json`         | a JSON String with JSON editor                     |
| `bin`          | a Buffer with editor                               |
| `binSimple`    | a Buffer without editor                            |
| `env`          | an environment variable                            |
| `node`         | a `node.` property expression                      |
| `cred`         | a secure credential                                |

```javascript
$(".input").typedInput({
    types: ["msg","string"]
});
```

#### `typeField`

*Type: CSS Selector*

In some circumstances it is desirable to already have an `<input>` element to store the type value of the `TypedInput`. This option allows such an existing element to be provided. As the type of the `TypedInput` is changed, the value of the provided input will also change.

```javascript
$(".input").typedInput({
    typeField: ".my-type-field"
});
```

### Methods

#### `disable()`

Disable the `TypedInput` when it is currently enabled.

```javascript
$(".input").typedInput('disable');
```

#### `disabled()`

Gets whether the `TypedInput` is currently disabled or not.

```javascript
$(".input").typedInput('disabled');
```

#### `hide()`

Hide the `TypedInput` when it is currently visible.

```javascript
$(".input").typedInput('hide');
```

#### `show`

Show the `TypedInput` when it is currently hidden.

```javascript
$(".input").typedInput('show');
```

#### `type()`

*Returns: String*

Gets the selected type of the `TypedInput`.

```javascript
var type = $(".input").typedInput('type');
```

#### `type(type)`

Sets the selected type of the `TypedInput`.

```javascript
$(".input").typedInput('type','msg');
```

#### `types(types)`

Sets the list of types offered by the `TypedInput`. See the description of the `types` option.

```javascript
$(".input").typedInput('types',['string','int']);
```

#### `validate()`

*Returns: Boolean*

Triggers a revalidation of the `TypedInput`'s type/value. This occurs automatically whenever the type or value change, but this method allows it to be run manually.

```javascript
var isValid = $(".input").typedInput('validate');
```

#### `value()`

*Returns: String*

Gets the value of the `TypedInput`.

```javascript
var value = $(".input").typedInput('value');
```

#### `value(value)`

Sets the value of the `TypedInput`.

```javascript
$(".input").typedInput('value','payload');
```

#### `width(width)`

Sets the width of the `TypedInput`. This must be used in place of the standard `jQuery.width()` function as it ensures the component resizes properly.

```javascript
$(".input").typedInput('width', '200px');
```

### Events

#### `change(type, value)`

Triggered when either the type or value of the input is changed.

```javascript
$(".input").on('change', function(type, value) {} );
```

### Types

#### `TypeDefinition`

A `TypeDefinition` object describes a type that can be offered by a `TypedInput` element.

It is an object with the following properties:

| Property   | Type     | Required | Description                                                  |
| ---------- | -------- | -------- | ------------------------------------------------------------ |
| `value`    | String   | yes      | The identifier for the type                                  |
| `label`    | String   |          | A label to display in the type menu                          |
| `icon`     | String   |          | An icon to display in the type menu                          |
| `options`  | Array    |          | If the type has a fixed set of values, this is an Array of String options for the value. For example, `["true","false"]` for the boolean type. |
| `hasValue` | Boolean  |          | Set to `false` if there is no value associated with the type. |
| `validate` | Function |          | A function to validate the value for the type.               |

##### Examples

###### Number type:

```javascript
{
    value:"num",
    label:"number",
    icon:"red/images/typedInput/09.png",
    validate:/^[+-]?[0-9]*\.?[0-9]*([eE][-+]?[0-9]+)?$/
}
```

###### Boolean type:

```javascript
{
    value:"bool",
    label:"boolean",
    icon:"red/images/typedInput/bool.png",
    options:["true","false"]
}
```

###### Timestamp type:

```javascript
{
    value:"date",
    label:"timestamp",
    hasValue:false
}
```

## EditableList

A replacement for a `<ol>` element where the items can be complex elements in their own right. Used by the core `Switch` and `Change` nodes.

TODO

### Options

#### `addButton`

*Type: Boolean | String*

#### `addItem(row, index, data)`

*Type: Function*

#### `connectWith`

*Type: CSS Selector

#### `header`

*Type: DOM/jQuery object

#### `height`

*Type: String | Number*

#### `filter(data)`

*Type: Function*

#### `resize`

*Type: Function*

#### `resizeItem(row, index)`

*Type: Function*

#### `scrollOnAdd`

*Type: Boolean*

#### `sort(itemDataA, itemDataB)`

*Type: Function*

#### `sortable`

*Type: Boolean | CSS Selector*

#### `sortItems(items)`

*Type: Function*

#### `removable`

*Type: Boolean*

#### `removeItem(data)`

*Type: Function*

## SearchBox

An enhanced `<input>` element that provides common features for a search input.

### Options

#### `delay`

*Type: Number*

Sets the delay, in ms, after the last keystroke before a `change` event is fired.

#### `minimumLength`

*Type: Number*

Sets the minimum length of the search string.

### Methods

#### `change()`

Trigger the change event on the search input.

```javascript
$(".input").searchBox('change');
```

#### `count(value)`

Sets the value of the count label on the search box. This can be used to provide feedback to the user as to how many ‘things’ the search currently matches. The `value` is a string.

The standard pattern to follow is:

- if the search box is empty, set it to the number of available items: `"300"`
- if the search box is not empty, set it to the number of matching items, as well as the number of available items: `"120 / 300"`

If `value` is null, undefined or blank, the count field is hidden.

```javascript
$(".input").searchBox('count', '120 / 300');
```

#### `value()`

*Returns: String*

Gets the current value of the search input.

```javascript
var type = $(".input").searchBox('value');
```

#### `value(value)`

Sets the current value of the search input.

```javascript
$(".input").searchBox('value','hello');
```

## TreeList

### Options

#### `data`

*Type: Array*

The initial data for the `TreeList`.

```javascript
[
    {
        label: 'Local',
        icon: 'fa fa-rocket',
        children: [
            { label: "child 1"},
            { label: "child 2"}
        ]
    }
]
```

Each item can have the following properties:

| Property   | Description                                                  |
| ---------- | ------------------------------------------------------------ |
| `label`    | The label for the item.                                      |
| `id`       | (optional) A unique identifier for the item                  |
| `class`    | (optional) A CSS class to apply to the item                  |
| `icon`     | (optional) A CSS class to apply as the icon, for example `"fa fa-rocket"`. |
| `selected` | (optional) if set, display a checkbox next to the item. Its state is set to the boolean value of this property |
| `children` | (optional) Identifies the child items of this one. Can be  provided as an array if the children are immediately known, or as a  function to get the children asynchronously. See below for details. |
| `expanded` | (optional) If the item has children, set whether to display the children |

If the `children` property is provided as a function, that function should accept a single argument of a callback function. That callback function should be called with the array of child items. This allows for the items to be retrieved asynchronously, such as via HTTP request.

```javascript
children: function(done) {
    $.getJSON('/some/url', function(result) {
        done(result);
    })
}
```

### Methods

#### `data()`

Returns the data the `TreeList` is displaying.

If any items had the `selected` property set on them, its value will reflect the current checkbox state.

#### `data(items)`

Sets the data to be displayed by the list.

See the `data` for details of the `items` argument.

```javascript
$(".input").treeList('data',[{label:"Colours"}]);
```

#### `empty()`

Removes all items from the list.

```javascript
$(".input").treeList('empty');
```

#### `show(itemId)`

Ensures an item is visible in the list. The argument `itemId` must correspond with the `id` property of an item in the list.

!!! note
    This currently only works for the topmost items in the list. It cannot be used to reveal items below the top level of the tree.

```javascript
$(".input").treeList('show','my-red-item');
```

### Events

#### `treelistselect(event, item)`

Triggered when an item is clicked. If the item had the `selected` property set originally, its value will be updated to reflect the state of the item’s checkbox.

```javascript
$(".input").on('treelistselect', function(event, item) {
    if (item.selected) {
        // The checkbox is checked
    } else {
        // The checkbox is not checked
    }
});
```

#### `treelistmouseout(event, item)`

Triggered when the mouse moves out of the item.

```javascript
$(".input").on('treelistmouseout', function(event, item) { });
```

#### `treelistmouseover(event, item)`

Triggered when the mouse moves over an item.

```javascript
$(".input").on('treelistmouseover', function(event, item) { });
```

