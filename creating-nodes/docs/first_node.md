# Creating your first node

This section is largely citing the Node-RED documentation.

Nodes get created when a flow is deployed, they may send and receive some messages whilst the flow is running and they get deleted when the next flow is deployed.

They consist at least of:

* one or more code files defining what the node does (the backend part)
* an html file that defines the node’s properties, edit dialog and help text (the frontend/UI part).

A `package.json` file is used to package it all together as a module.

[TOC]

## Creating a simple node in PHP

This example will show how to create a node in PHP that converts message payloads to all lower-case characters.

Create a directory named `node-blue-node-example-lower-case` (the directory name must match the module name) where you will develop your code. Within that directory, create the following files:

- `package.json`
- `lower-case.php`
- `lower-case.html`

#### :fa-file: package.json

This is a standard file used by Node-BLUE modules to describe their contents.

Add the following content:

```json
{
  "name": "node-blue-node-example-lower-case",
  "version": "1.0.0",
  "description": "A node to convert strings to lower case.",
  "homepage": "https://lower-case.example.com",
  "node-blue": {
    "maxThreadCounts": {
      "lower-case": 0
    },
    "nodes": {
      "lower-case": "lower-case.php"
    }
  }
}
```

This tells the runtime the name of the module and what node files the module contains. `maxThreadCounts` is needed only, when the node starts new threads and is relevant to calculate the number of nodes that can run within one process before reaching the thread limit.

For more information about how to package your node, including requirements on naming and other properties that should be set before publishing your node, refer to the packaging guide (TODO).

**Note**: Please do ***not\*** publish this example node!

#### :fa-file: lower-case.php

```php
<?php
declare(strict_types=1);

class HomegearNode extends HomegearNodeBase
{
    private $hg = null;

    public function __construct()
    {
	    $this->hg = new \Homegear\Homegear();
    }

    public function input(array $nodeInfo, int $inputIndex, array $msg)
    {
        $msg['payload'] = strtolower($msg['payload']);
        $this->output(0, $msg);
    }
}
```

The node is a PHP class derived from `HomegearNodeBase`. This type of node is called "simple PHP node". The object is newly created by the runtime everytime a message arrives at the node. The runtime then executes the `input` function. Three parameters are passed:

1. `$nodeInfo`: Information about the node (like `name`, `id`, ...)
2. `$inputIndex`: The index of the input the message arrived at
3. `$msg`: The message object

Within `input`, the node changes the payload to lower case, then calls the `output` function to pass the message on in the flow.

For more information about the runtime part of the node, see here (TODO).

#### :fa-file: lower-case.html

```html
<script type="text/javascript">
    RED.nodes.registerType('lower-case',{
        category: 'function',
        color: '#a6bbcf',
        defaults: {
            name: {value:""}
        },
        inputs:1,
        outputs:1,
        icon: "file.png",
        label: function() {
            return this.name||"lower-case";
        }
    });
</script>

<script type="text/html" data-template-name="lower-case">
    <div class="form-row">
        <label for="node-input-name"><i class="fa fa-tag"></i> Name</label>
        <input type="text" id="node-input-name" placeholder="Name">
    </div>
</script>

<script type="text/html" data-help-name="lower-case">
    <p>A simple node that converts the message payloads into all lower-case characters</p>
</script>
```

A node’s HTML file provides the following things:

- the main node definition that is registered with the editor
- the edit template
- the help text

In this example, the node has a single editable property, `name`. Whilst not required, there is a widely used convention to this property to help distinguish between multiple instances of a node in a single flow.

For more information about the editor part of the node, see here (TODO).

### Testing your PHP node in Node-BLUE

Once you have created a basic node module as described above, you can install it into your Node-BLUE runtime.

To test a node module locally, create a link to the folder containing the files in your Node-BLUE node directory (by default `/var/lib/homegear/node-blue/nodes`). For example if your node is located at `~/dev/node-blue-node-example-lower-case` you would do the following:

```bash
cd /var/lib/homegear/node-blue/nodes
ln -s ~/dev/node-blue-node-example-lower-case node-blue-node-example-lower-case
```

This creates a symbolic link to your node module project directory in  `/var/lib/homegear/node-blue/nodes` so that Node-BLUE will discover the node when it starts or reloads. Any changes to  the node’s HTML file can be picked up by simply reloading the Node-BLUE UI. Changes to the code files or locales require a Node-BLUE restart. To restart Node-BLUE without restarting the whole Homegear process, execute the following command in your shell:

```
homegear -e fr
```

or select `Restart Node-BLUE` from the Node-BLUE UI's menu.

### Unit testing with PHP

You can do unit tests with the help of Homegear's RPC methods in combination with the node `unit-test-helper`. This special node makes Homegear save all arriving messages.

Using the RPC methods and the `unit-test-helper` node you can create test flows, and then assert that the output is working as expected. For example, to add a unit test to the lower-case node you can add a `test` folder to your node module package containing a file called `lower-case_spec.php`. Placing the file here with this filename enables running automatic unit tests.

#### :fa-file: test/lower-case_spec.php

```php
<?php
$hg = new \Homegear\Homegear();

//A flow definition
$testFlow = [
	[
		'id' => 'n1', //Is replaced by addNodesToFlow below
		'type' => 'lower-case',
		'wires' => [
			[['id' => 'n2','port' => 0]] //Output 1 => Input 1
		]
	],
	[
		'id' => 'n2', //Is replaced by addNodesToFlow below
		'type' => 'unit-test-helper',
		'inputs' => 1
	]
];

//Init - add test flow to Node-BLUE
$nodeIds = $hg->addNodesToFlow('Unit test' /* tab */, 'unit-test' /* tag */, $testFlow);
if ($nodeIds === false) die('Error =>  Could not create flow.');
$n1 = $nodeIds['n1']; //Get node ID of inserted lower-case node
$n2 = $nodeIds['n2']; //Get node ID of inserted helper node
//Trigger a Node-BLUE restart
if ($hg->restartFlows() !== true) die('Error => Could not restart flows.');
while (!$hg->nodeBlueIsReady()) {
    //Wait for Node-BLUE to become ready again
	sleep(1);
}

//{{{ Perform the actual tests
$hg->setNodeVariable($n1, 'fixedInput0', ['UpperCase']);
sleep(1); //Wait for asynchronous processing to finish
//This returns up to the last 10 input values. The latest value is at index 0.
$inputHistory = $hg->getNodeVariable($n2, 'inputHistory0');
assert(count($inputHistory) === 1, new AssertionError('No message was passed on.'));
assert($inputHistory[0][1]['payload'] === 'uppercase', new AssertionError('Payload is not lower case.'));
//}}}

//Clean up
$hg->removeNodesFromFlow('Unit test', 'unit-test');
```

These tests check to see that the node is loaded into the runtime correctly, and that it correctly changes the payload to lower case as expected.

This test file can be executed using:

```bash
homegear -e rs lower-case_spec.php
```

## Creating a simple node in JavaScript

This example will show how to create a node in JavaScript that converts message payloads to all lower-case characters. JavaScript nodes are executed in a separate process (one process for all JavaScript nodes) using Node.js which is compiled into Homegear.

Create a directory named `node-blue-node-example-lower-case` (the directory name must match the module name) where you will develop your code. Within that directory, create the following files:

- `package.json`
- `lower-case.js`
- `lower-case.html`

#### :fa-file: package.json

This is a standard file used by Node-BLUE modules to describe their contents.

Add the following content:

```json
{
  "name": "node-blue-node-example-lower-case",
  "version": "1.0.0",
  "description": "A node to convert strings to lower case.",
  "homepage": "https://lower-case.example.com",
  "node-blue": {
    "nodes": {
      "lower-case": "lower-case.js"
    }
  }
}
```

This tells the runtime the name of the module and what node files the module contains.

!!! note
    In contrast to PHP nodes no `maxThreadCounts` property is needed as JavaScript nodes don't start threads.

For more information about how to package your node, including requirements on naming and other properties that should be set before publishing your node, refer to the packaging guide (TODO).

**Note**: Please do **\*not\*** publish this example node!

#### :fa-file: lower-case.js

```js
module.exports = function(RED) {
    function LowerCaseNode(config) {
        RED.nodes.createNode(this,config);
        var node = this;
        node.on('input', function(msg) {
            msg.payload = msg.payload.toLowerCase();
            node.send(msg);
        });
    }
    RED.nodes.registerType("lower-case",LowerCaseNode);
}
```

The node is wrapped as a Node.js module. The module exports a function that gets called when the runtime loads the node on start-up. The function is called with a single argument, `RED`, that provides the module access to the Node-RED runtime api.

The node itself is defined by a function, `LowerCaseNode` that gets called whenever a new instance of the node is created. It is passed an object containing the node-specific properties set in the flow editor.

The function calls the `RED.nodes.createNode` function to initialize the features shared by all nodes. After that, the node-specific code lives.

In this instance, the node registers a listener to the `input` event which gets called whenever a message arrives at the node. Within this listener, it changes the payload to lower case, then calls the `send` function to pass the message on in the flow.

!!! note
    In contrast to Node-RED, Node-BLUE supports multiple inputs. The index of the input the message arrived at is stored in the property `msg.inputIndex`.

Finally, the `LowerCaseNode` function is registered with the runtime using the name for the node, `lower-case`.

If the node has any external module dependencies, they must be included in the `dependencies` section of its `package.json` file.

For more information about the runtime part of the node, see here (TODO).

#### :fa-file: lower-case.html

```html
<script type="text/javascript">
    RED.nodes.registerType('lower-case',{
        category: 'function',
        color: '#a6bbcf',
        defaults: {
            name: {value:""}
        },
        inputs:1,
        outputs:1,
        icon: "file.png",
        label: function() {
            return this.name||"lower-case";
        }
    });
</script>

<script type="text/html" data-template-name="lower-case">
    <div class="form-row">
        <label for="node-input-name"><i class="fa fa-tag"></i> Name</label>
        <input type="text" id="node-input-name" placeholder="Name">
    </div>
</script>

<script type="text/html" data-help-name="lower-case">
    <p>A simple node that converts the message payloads into all lower-case characters</p>
</script>
```

A node’s HTML file provides the following things:

- the main node definition that is registered with the editor
- the edit template
- the help text

In this example, the node has a single editable property, `name`. Whilst not required, there is a widely used convention to this property to help distinguish between multiple instances of a node in a single flow.

For more information about the editor part of the node, see here (TODO).

### Testing your JavaScript node in Node-BLUE

Once you have created a basic node module as described above, you can install it into your Node-BLUE runtime.

In contrast to nodes written in other languages, JavaScript nodes are stored in a separate directory for Node-RED compatibility. By default this is `/var/lib/homegear/node-blue/nodes/node-red-nodes`.

To test a node module locally the [`npm install <folder>`](https://docs.npmjs.com/cli/install) command can be used. This allows you to develop the node in a local directory and have it linked into a local Node-BLUE install during development.

In your node directory user directory, run:

```bash
npm install <location of node module>
```

This makes the node visible to Homegear's Node-RED instance (we call it Node-PINK). To make it visible to Node-BLUE as well, a soft link needs to be created from Node-BLUE's node directory to the node's location:

```
cd /var/lib/homegear/node-blue/nodes
ln -s <node directory> <module name>
```

For example, if your node is located at `~/dev/node-blue-node-example-lower-case` you would do the following:

```bash
cd /var/lib/homegear/node-blue/nodes/node-red-nodes
npm install ~/dev/node-blue-node-example-lower-case
cd /var/lib/homegear/node-blue/nodes
ln -s node-red-nodes/node-blue-node-example-lower-case node-blue-node-example-lower-case
```

Any changes to  the node’s HTML file can be picked up by simply reloading the Node-BLUE UI. Changes to the code files or locales require a Node-BLUE restart. To restart Node-BLUE without restarting the whole Homegear process, execute the following command in your shell:

```
homegear -e fr
```

or select `Restart Node-BLUE` from the Node-BLUE UI's menu.

### Unit testing with JavaScript

You can do unit tests with the help of Homegear's RPC methods in combination with the node `unit-test-helper`. This special node makes Homegear save all arriving messages.

Using the RPC methods and the `unit-test-helper` node you can create test flows, and then assert that the output is working as expected. For example, to add a unit test to the lower-case node you can add a `test` folder to your node module package containing a file called `lower-case_spec.js`. Placing the file here with this filename enables running automatic unit tests.

#### :fa-file: test/lower-case_spec.js

```javascript
'use strict'
var homegear = require('@homegear/homegear-nodejs');
const assert = require('assert').strict;

var initialized = false;
var nodeIds;

function nodeBlueIsReady() {
    if (hg.invoke('nodeBlueIsReady')) initialized = true;
    else setTimeout(nodeBlueIsReady, 1000);
}

function connected() {
    addTestFlow();
}

function cleanUp() {
    hg.invoke('removeNodesFromFlow', ['Unit test', 'unit-test']);
}

function main() {
    if (!initialized) {
        setTimeout(main, 1000);
        return;
    }

    unittest();
}

function addTestFlow() {
    var testFlow = [
        {
            id: 'n1', //Is replaced by addNodesToFlow below
            type: 'lower-case',
            wires: [
                [{id: 'n2', port: 0}] //Output 1 => Input 1
            ]
        },
        {
            id: 'n2', //Is replaced by addNodesToFlow below
            type: 'unit-test-helper',
            inputs: 1
        }
    ];

    //Init - add test flow to Node-BLUE
    nodeIds = hg.invoke('addNodesToFlow', ['Unit test' /* tab */, 'unit-test' /* tag */, testFlow]);
    if (nodeIds === false) throw new Error('Error =>  Could not create flow.');
    //Trigger a Node-BLUE restart
    if (hg.invoke('restartFlows') !== true) throw new Error('Error => Could not restart flows.');
    setTimeout(nodeBlueIsReady, 1000);
}

function unittest() {
    var n1 = nodeIds.n1; //Get node ID of inserted lower-case node
    var n2 = nodeIds.n2; //Get node ID of inserted helper node

    //{{{ Perform the actual tests
    hg.invoke('setNodeVariable', [n1, 'fixedInput0', {'payload': 'UpperCase'}]);
    
    setTimeout(function() { //Wait for asynchronous processing to finish
        //This returns up to the last 10 input values. The latest value is at index 0.
        var inputHistory = hg.invoke('getNodeVariable', [n2, 'inputHistory0']);
        assert.equal(inputHistory.length, 1);
        assert.equal(inputHistory[0][1].payload, 'uppercase');

        cleanUp();
    }, 1000);
    //}}}
}

var hg = new homegear.Homegear('/var/lib/homegear/homegearIPC.sock', connected);

setTimeout(main, 1000);
```

These tests check to see that the node is loaded into the runtime correctly, and that it correctly changes the payload to lower case as expected.

This test file can be executed using:

```bash
homegear-node lower-case_spec.js
```

## Creating a simple node in Python

This example will show how to create a node in Python that converts message payloads to all lower-case characters.

Create a directory named `node-blue-node-example-lower-case` (the directory name must match the module name) where you will develop your code. Within that directory, create the following files:

- `package.json`
- `lower-case.py`
- `lower-case.html`

#### :fa-file: package.json

This is a standard file used by Node-BLUE modules to describe their contents.

Add the following content:

```json
{
  "name": "node-blue-node-example-lower-case",
  "version": "1.0.0",
  "description": "A node to convert strings to lower case.",
  "homepage": "https://lower-case.example.com",
  "node-blue": {
    "maxThreadCounts": {
      "lower-case": 0
    },
    "nodes": {
      "lower-case": "lower-case.php"
    }
  }
}
```

This tells the runtime the name of the module and what node files the module contains. `maxThreadCounts` is needed only, when the node starts new threads and is relevant to calculate the number of nodes that can run within one process before reaching the thread limit.

For more information about how to package your node, including requirements on naming and other properties that should be set before publishing your node, refer to the packaging guide (TODO).

**Note**: Please do ***not\*** publish this example node!

#### :fa-file: lower-case.php

```php
<?php
declare(strict_types=1);

class HomegearNode extends HomegearNodeBase
{
    private $hg = null;

    public function __construct()
    {
	    $this->hg = new \Homegear\Homegear();
    }

    public function input(array $nodeInfo, int $inputIndex, array $msg)
    {
        $msg['payload'] = strtolower($msg['payload']);
        $this->output(0, $msg);
    }
}
```

The node is a PHP class derived from `HomegearNodeBase`. This type of node is called "simple PHP node". The object is newly created by the runtime everytime a message arrives at the node. The runtime then executes the `input` function. Three parameters are passed:

1. `$nodeInfo`: Information about the node (like `name`, `id`, ...)
2. `$inputIndex`: The index of the input the message arrived at
3. `$msg`: The message object

Within `input`, the node changes the payload to lower case, then calls the `output` function to pass the message on in the flow.

For more information about the runtime part of the node, see here (TODO).

#### :fa-file: lower-case.html

```html
<script type="text/javascript">
    RED.nodes.registerType('lower-case',{
        category: 'function',
        color: '#a6bbcf',
        defaults: {
            name: {value:""}
        },
        inputs:1,
        outputs:1,
        icon: "file.png",
        label: function() {
            return this.name||"lower-case";
        }
    });
</script>

<script type="text/html" data-template-name="lower-case">
    <div class="form-row">
        <label for="node-input-name"><i class="fa fa-tag"></i> Name</label>
        <input type="text" id="node-input-name" placeholder="Name">
    </div>
</script>

<script type="text/html" data-help-name="lower-case">
    <p>A simple node that converts the message payloads into all lower-case characters</p>
</script>
```

A node’s HTML file provides the following things:

- the main node definition that is registered with the editor
- the edit template
- the help text

In this example, the node has a single editable property, `name`. Whilst not required, there is a widely used convention to this property to help distinguish between multiple instances of a node in a single flow.

For more information about the editor part of the node, see here (TODO).

### Testing your PHP node in Node-BLUE

Once you have created a basic node module as described above, you can install it into your Node-BLUE runtime.

To test a node module locally, create a link to the folder containing the files in your Node-BLUE node directory (by default `/var/lib/homegear/node-blue/nodes`). For example if your node is located at `~/dev/node-blue-node-example-lower-case` you would do the following:

```bash
cd /var/lib/homegear/node-blue/nodes
ln -s ~/dev/node-blue-node-example-lower-case node-blue-node-example-lower-case
```

This creates a symbolic link to your node module project directory in  `/var/lib/homegear/node-blue/nodes` so that Node-BLUE will discover the node when it starts or reloads. Any changes to  the node’s HTML file can be picked up by simply reloading the Node-BLUE UI. Changes to the code files or locales require a Node-BLUE restart. To restart Node-BLUE without restarting the whole Homegear process, execute the following command in your shell:

```
homegear -e fr
```

or select `Restart Node-BLUE` from the Node-BLUE UI's menu.

### Unit testing with PHP

You can do unit tests with the help of Homegear's RPC methods in combination with the node `unit-test-helper`. This special node makes Homegear save all arriving messages.

Using the RPC methods and the `unit-test-helper` node you can create test flows, and then assert that the output is working as expected. For example, to add a unit test to the lower-case node you can add a `test` folder to your node module package containing a file called `lower-case_spec.php`. Placing the file here with this filename enables running automatic unit tests.

#### :fa-file: test/lower-case_spec.php

```php
<?php
$hg = new \Homegear\Homegear();

//A flow definition
$testFlow = [
	[
		'id' => 'n1', //Is replaced by addNodesToFlow below
		'type' => 'lower-case',
		'wires' => [
			[['id' => 'n2','port' => 0]] //Output 1 => Input 1
		]
	],
	[
		'id' => 'n2', //Is replaced by addNodesToFlow below
		'type' => 'unit-test-helper',
		'inputs' => 1
	]
];

//Init - add test flow to Node-BLUE
$nodeIds = $hg->addNodesToFlow('Unit test' /* tab */, 'unit-test' /* tag */, $testFlow);
if ($nodeIds === false) die('Error =>  Could not create flow.');
$n1 = $nodeIds['n1']; //Get node ID of inserted lower-case node
$n2 = $nodeIds['n2']; //Get node ID of inserted helper node
//Trigger a Node-BLUE restart
if ($hg->restartFlows() !== true) die('Error => Could not restart flows.');
while (!$hg->nodeBlueIsReady()) {
    //Wait for Node-BLUE to become ready again
	sleep(1);
}

//{{{ Perform the actual tests
$hg->setNodeVariable($n1, 'fixedInput0', ['UpperCase']);
sleep(1); //Wait for asynchronous processing to finish
//This returns up to the last 10 input values. The latest value is at index 0.
$inputHistory = $hg->getNodeVariable($n2, 'inputHistory0');
assert(count($inputHistory) === 1, new AssertionError('No message was passed on.'));
assert($inputHistory[0][1]['payload'] === 'uppercase', new AssertionError('Payload is not lower case.'));
//}}}

//Clean up
$hg->removeNodesFromFlow('Unit test', 'unit-test');
```

These tests check to see that the node is loaded into the runtime correctly, and that it correctly changes the payload to lower case as expected.

This test file can be executed using:

```bash
homegear -e rs lower-case_spec.php
```