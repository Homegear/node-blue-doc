# Creating your first node

This section is largely citing the Node-RED documentation.

Nodes get created when a flow is deployed, they may send and receive some messages whilst the flow is running and they get deleted when the next flow is deployed.

They consist at least of:

* one or more code files defining what the node does (the backend part)
* an html file that defines the node’s properties, edit dialog and help text (the frontend/UI part).

A `package.json` file is used to package it all together as a module.

## Creating a simple node in PHP

### Implementing the node

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

### Implementing the node

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
    In contrast to PHP and C++ nodes no `maxThreadCounts` property is needed as JavaScript nodes don't start threads.

For more information about how to package your node, including requirements on naming and other properties that should be set before publishing your node, refer to the packaging guide (TODO).

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

### Implementing the node

This example will show how to create a node in Python that converts message payloads to all lower-case characters. Python nodes are executed in a separate process (one process for each Python node) using the system Python binary.

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
    "nodes": {
      "lower-case": "lower-case.py"
    }
  }
}
```

This tells the runtime the name of the module and what node files the module contains.

!!! note
    In contrast to PHP and C++ nodes no `maxThreadCounts` property is needed as Python nodes run in a seperate process.

For more information about how to package your node, including requirements on naming and other properties that should be set before publishing your node, refer to the packaging guide (TODO).

#### :fa-file: lower-case.py

```python
from homegear import Homegear
import threading
import sys
import asyncio

loop = asyncio.get_event_loop()
nodeInfo = None
hg = None

startUpComplete = threading.Condition()

# This callback method is called on Homegear variable changes.
def eventHandler(eventSource, peerId, channel, variableName, value):
	# Note that the event handler is called by a different thread than the main thread. I. e. thread synchronization is
	# needed when you access non local variables. We use a condition variable for startup and "run_coroutine_threadsafe()"
	# for variable events.
	
	# When the flows are fully started, "startUpComplete" is set to "true". Wait for this event.
	if eventSource == "nodeBlue" and peerId == 0 and variableName == "startUpComplete":
		startUpComplete.acquire()
		global nodeInfo
		nodeInfo = value
		startUpComplete.notify()
		startUpComplete.release()
	else:
		asyncio.run_coroutine_threadsafe(eventHandlerThreadSafe(eventSource, peerId, channel, variableName, value), loop).result()
	
async def eventHandlerThreadSafe(eventSource, peerId, channel, variableName, value):
	# You don't have to worry about threads here and can safely access all variables.
	pass

# This callback method is called when a message arrives on one of the node's inputs.
def nodeInput(nodeInfo, inputIndex, message):
	# Note that the node input handler is called by a different thread than the main thread. I. e. thread synchronization is
	# needed here. We use "run_coroutine_threadsafe()".
	asyncio.run_coroutine_threadsafe(nodeInputThreadSafe(nodeInfo, inputIndex, message), loop).result()

async def nodeInputThreadSafe(nodeInfo, inputIndex, msg):
	# You don't have to worry about threads here and can safely access all variables.
	hg.nodeOutput(0, {"payload": message.payload.lower()})

async def appstart(loop, hg):
	while hg.connected():
		await asyncio.sleep(1)
	return 0

# hg waits until the connection is established (but for a maximum of 2 seconds).
# The socket path is passed in sys.argv[1], the node's ID in sys.argv[2]
hg = Homegear(sys.argv[1], eventHandler, sys.argv[2], nodeInput);

# Wait for the flows to start.
startUpComplete.acquire()
while hg.connected():
	if startUpComplete.wait(1) == True:
		break
startUpComplete.release()

# The node is now fully started. Start event loop.
loop.run_until_complete(appstart(loop, hg))
```

This node is a normal Python script importing the Homegear module. It uses Unix Domain Sockets (or IPC) to communicate with Homegear. This is why we need to wait until the node is connected to Homegear and Node-BLUE is started before executing any Homegear-specific methods.

The script does the following:

1. It creates a new Homegear object. There are two callback methods passed to the object: `eventHandler` which is called on Homegear variable changes and `nodeInput` which is called whenever a new message arrives.
2. We wait until we get the `startUpComplete` event from Node-BLUE. The event is sent to the `eventHandler` callback method. In this case we wait for the event in the main thread by using a condition variable. Once the event arrives, the condition variable is notified and the main thread continues.
3. We use asyncio in this example (but you don't have to). So after Node-BLUE is ready we start the main loop.
4. Now we are ready to process events from Homegear in `nodeInput` or `eventHandler`. In both callback functions we just call the thread safe variant where we can work with variables without having to worry about thread synchronisation.
5. In `appstart()` we check, if we are still connected to Homegear. If not, the script exits.

Whenever a message arrives, the `nodeInput` function is executed. Three parameters are passed:

1. `nodeInfo`: Information about the node (like `name`, `id`, ...)
2. `inputIndex`: The index of the input the message arrived at
3. `msg`: The message object

Within `input`, the node changes the payload to lower case, then calls the `nodeOutput` function to pass the message on in the flow.

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

### Testing your Python node in Node-BLUE

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

### Unit testing with Python

You can do unit tests with the help of Homegear's RPC methods in combination with the node `unit-test-helper`. This special node makes Homegear save all arriving messages.

Using the RPC methods and the `unit-test-helper` node you can create test flows, and then assert that the output is working as expected. For example, to add a unit test to the lower-case node you can add a `test` folder to your node module package containing a file called `lower-case_spec.php`. Placing the file here with this filename enables running automatic unit tests.

#### :fa-file: test/lower-case_spec.py

```python
from homegear import Homegear
import sys
import time

# hg waits until the connection is established (but for a maximum of 2 seconds).
# The socket path is passed in sys.argv[1]
hg = Homegear(sys.argv[1])

testFlow = [
    {
        "id": "n1", # Is replaced by addNodesToFlow below
        "type": "lower-case",
        "wires": [
            [{"id": "n2", "port": 0}] # Output 1 => Input 1
        ]
    },
    {
        "id": "n2", # Is replaced by addNodesToFlow below
        "type": "unit-test-helper",
        "inputs": 1
    }
]

nodeIds = hg.addNodesToFlow("Unit test", "unit-test", testFlow)
if nodeIds == False:
    raise SystemError('Error =>  Could not create flow.')
n1 = nodeIds["n1"] # Get node ID of inserted lower-case node
n2 = nodeIds["n2"] # Get node ID of inserted helper node
# Trigger a Node-BLUE restart
if hg.restartFlows() != True:
    raise SystemError("Error => Could not restart flows.")
while not hg.nodeBlueIsReady():
    # Wait for Node-BLUE to become ready again
    time.sleep(1)

# {{{ Perform the actual tests
hg.setNodeVariable(n1, "fixedInput0", {"payload": "UpperCase"})
time.sleep(1) # Wait for asynchronous processing to finish
# This returns up to the last 10 input values. The latest value is at index 0.
inputHistory = hg.getNodeVariable(n2, "inputHistory0")
assert len(inputHistory) == 1, "No message was passed on."
assert inputHistory[0][1]['payload'] == "uppercase", "Payload is not lower case."
# }}}

# Clean up
hg.removeNodesFromFlow("Unit test", "unit-test")
```

These tests check to see that the node is loaded into the runtime correctly, and that it correctly changes the payload to lower case as expected.

This test file can be executed using:

```bash
python3 lower-case_spec.py
```

## Creating a simple node in C++

This example will show how to create a node in C++ that converts message payloads to all lower-case characters.

Create a directory named node-blue-node-example-lower-case (the directory name must match the module name) where you will develop your code. Within that directory, create the following files:

    package.json
    LowerCase.h
    LowerCase.cpp
    CMakeLists.txt
    lower-case.html

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
      "lower-case": "lower-case.so"
    }
  }
}
```

This tells the runtime the name of the module and what node files the module contains. `maxThreadCounts` is needed only, when the node starts new threads and is relevant to calculate the number of nodes that can run within one process before reaching the thread limit.

For more information about how to package your node, including requirements on naming and other properties that should be set before publishing your node, refer to the packaging guide (TODO).

#### :fa-file: LowerCase.h

```c++
#ifndef LOWER_CASE_H_
#define LOWER_CASE_H_

#include <homegear-node/NodeFactory.h>
#include <homegear-node/INode.h>

class MyFactory : Flows::NodeFactory {
 public:
  Flows::INode *createNode(const std::string &path, const std::string &type, const std::atomic_bool *frontendConnected) override;
};

extern "C" Flows::NodeFactory *getFactory();

namespace LowerCase {

class LowerCase : public Flows::INode {
 public:
  LowerCase(const std::string &path, const std::string &type, const std::atomic_bool *frontendConnected);
  ~LowerCase() override;

  void input(const Flows::PNodeInfo &info, uint32_t index, const Flows::PVariable &message) override;
 private:
};

}

#endif
```



#### :fa-file: LowerCase.cpp

```c++
#include "LowerCase.h"

Flows::INode *MyFactory::createNode(const std::string &path, const std::string &type, const std::atomic_bool *frontendConnected) {
  return new LowerCase::LowerCase(path, type, frontendConnected);
}

Flows::NodeFactory *getFactory() {
  return (Flows::NodeFactory *)(new MyFactory);
}

namespace LowerCase {

LowerCase::LowerCase(const std::string &path, const std::string &type, const std::atomic_bool *frontendConnected) : Flows::INode(path, type, frontendConnected) {
}

LowerCase::~LowerCase() = default;

void LowerCase::input(const Flows::PNodeInfo &info, uint32_t index, const Flows::PVariable &message) {
  try {
    auto newMessage = std::make_shared<Flows::Variable>(Flows::VariableType::tStruct);
    *newMessage = *message; //We are not allowed to change message, so we create a copy.
    auto &s = newMessage->structValue->at("payload")->stringValue;
    std::transform(s.begin(), s.end(), s.begin(), ::tolower);
    output(0, newMessage);
  }
  catch (const std::exception &ex) {
    _out->printEx(__FILE__, __LINE__, __PRETTY_FUNCTION__, ex.what());
  }
}

}
```

The code above is loaded by Homegear and executed by the Node-BLUE process. The NodeFactory is needed by Homegear to dynamically load the node. This part can just be copied into each node. 

The node itself is implemented as a class `LowerCase` derived from `Flows::INode`. It overrides the method `input` which gets called whenever a message arrives at the node. Three parameters are passed to `input`:

1. `info`: Information about the node (like `name`, `id`, ...)
2. `index`: The index of the input the message arrived at
3. `message`: The message object

In this method the payload is changed to lower case. Then `output()` is called to pass the message on in the flow.

For more information about the runtime part of the node, see here (TODO).

#### :fa-file: CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.16)
project(lower-case)
set(CMAKE_CXX_STANDARD 17)
include_directories(.)
add_library(lower-case SHARED LowerCase.cpp LowerCase.h)
add_custom_command(TARGET lower-case POST_BUILD COMMAND mv ARGS liblower-case.so ../lower-case.so)
```

CMake is used by Node-BLUE to compile nodes from the palette, so a CMakeLists.txt file needs to be provided. The line worth mentioning is the last line which renames the library file and copies it to the correct location.

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

### Compiling your C++ node

To compile your C++ node execute the following:

```bash
cd <node directory>
mkdir build
cd build
cmake ..
make -j
```



### Testing your C++ node in Node-BLUE

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

### Unit testing with Python

Unit testing can be done with C++ by using the same methods as with the other programming languages. It is recommended though to use a scripting language.