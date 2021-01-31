# Code files of Python nodes

Python nodes use the Python executable installed on the system. They connect to Node-BLUE using Unix domain sockets (or IPC sockets). Like this, Node-BLUE has full Python support including all Python modules.

## Homegear object

To connect to Node-BLUE we need to import the Homegear module and create a Homegear object:

```python
from homegear import Homegear
import sys

socketPath = sys.argv[1]
nodeId = sys.argv[2]

# hg waits until the connection is established (but for a maximum of 2 seonds).
# The socket path is passed in sys.argv[1], the node's ID in sys.argv[2]
hg = Homegear(socketPath, eventHandler, nodeId, nodeInput)
```

As IPC sockets are used, we need to know the path to the socket. We also need to know the node's ID. Node-BLUE passes the socket path in `sys.argv[1]` and the node's ID in `sys.argv[2]`.

In addition we need two callback methods: `eventHandler` and `nodeInput`. The first one is called on any variable update within Homegear (see the [Homegear reference](https://ref.homegear.eu/rpc.html#affixSection22)). `eventHandler` also is called when Node-BLUE is fully started. `nodeInput` is called, when a new message arrives. Our example now looks like this:

```python
from homegear import Homegear
import sys

socketPath = sys.argv[1]
nodeId = sys.argv[2]

def eventHandler(eventSource, peerId, channel, variableName, value):
    pass

def nodeInput(nodeInfo, inputIndex, msg):
    pass

# hg waits until the connection is established (but for a maximum of 2 seonds).
# The socket path is passed in sys.argv[1], the node's ID in sys.argv[2]
hg = Homegear(socketPath, eventHandler, nodeId, nodeInput)
```

Now we just need to make sure, the Python process keeps running. The recommended way to do so is to check if the Homegear object is still connected to Homegear. Our full Python node template then looks like this:

### Python node template

```python
from homegear import Homegear
import sys
import time

socketPath = sys.argv[1]
nodeId = sys.argv[2]

# This callback method is called on Homegear variable changes.
def eventHandler(eventSource, peerId, channel, variableName, value):
    pass

# This callback method is called when a message arrives on one of the node's inputs.
def nodeInput(nodeInfo, inputIndex, msg):
    pass

# hg waits until the connection is established (but for a maximum of 2 seonds).
# The socket path is passed in sys.argv[1], the node's ID in sys.argv[2]
hg = Homegear(socketPath, eventHandler, nodeId, nodeInput)

while hg.connected():
	time.sleep(1)
```

!!! note
    See [Using asyncio for thread synchronization](#using-asyncio-for-thread-synchronization) for a template with thread synchronization.

## Start and stop

Every individual Python node is executed in it's own Python process. The node is started in the  `start` phase of the [initialization process](node_initialization_and_deinitialization.md) and stopped in the `stop` phase of the [deinitialization process](node_initialization_and_deinitialization.md). `init` is not available. After the signal 15 is received, the Python process should exit within 30 seconds.

### Wait for Node-BLUE start to complete

In many cases it is required to know when Node-BLUE start up is complete. For this Node-BLUE sends an event to the Python process and `eventHandler` is called with `eventSource` set to `nodeBlue`, `peerId` set to `0`, `variableName` set to `startUpComplete` and `value` set to the node info structure. The challenge now is to notify the main thread about this event as `eventHandler` runs in a different thread. For this thread synchronization techniques must be used. In our example we use a condition variable:

```python
from homegear import Homegear
import threading
import sys
import time

hg = None
nodeInfo = None
socketPath = sys.argv[1]
nodeId = sys.argv[2]
startUpComplete = threading.Condition()

# This callback method is called on Homegear variable changes.
def eventHandler(eventSource, peerId, channel, variableName, value):	
	# When the flows are fully started, "startUpComplete" is set to "true". Wait for this event.
	if eventSource == "nodeBlue" and peerId == 0 and variableName == "startUpComplete":
		startUpComplete.acquire()
        global nodeInfo
		nodeInfo = value
		startUpComplete.notify()
		startUpComplete.release()
	

# This callback method is called when a message arrives on one of the node's inputs.
def nodeInput(nodeInfo, inputIndex, msg):
	pass

# hg waits until the connection is established (but for a maximum of 2 seonds).
# The socket path is passed in sys.argv[1], the node's ID in sys.argv[2]
hg = Homegear(socketPath, eventHandler, nodeId, nodeInput)

# Wait for the flows to start.
startUpComplete.acquire()
while hg.connected():
	if startUpComplete.wait(1) == True:
		break
startUpComplete.release()

# The node is now fully started.

# The script needs to run permanently. Just stop it, when Homegear is not running anymore.
# Homegear sends a signal 15 (SIGTERM) to the process to stop it.
while hg.connected():
	time.sleep(1)

```

## Receiving messages

Whenever a new message arrives `nodeInput` is executed:

```python
def nodeInput(nodeInfo, inputIndex, msg):
    # Do something with 'msg'
	pass
```

!!! warning
    `nodeInput` is executed in it's own thread. To communicate with the main thread, thread synchronization is required.

### Multiple inputs

If the node has more than one input, the input the message arrived at can be read from `inputIndex`.

### Thread synchronization

We recommend to use `asyncio`. See the section about [asyncio](#using-asyncio-for-thread-synchronization).

## Handling errors and logging events

If the node encounters an error whilst handling the message or if it wants to write something to the log file, it should call the `nodeLog` method:

```python
logLevel = 2
message = "Something really bad happened"
hg.nodeLog(logLevel, message)
```

Available log levels are:

| Log level | Description |
| --------- | ----------- |
| 1         | Critical    |
| 2         | Error       |
| 3         | Warning     |
| 4         | Info        |
| 5         | Debug       |

The log message is written to Homegear's flows log file. Warnings and errors also trigger `Catch` nodes and - if not handled by a `Catch` node - are written to debug tabs of connected frontends.

## Calling Homegear RPC methods

For an overview of the available RPC methods, visit [Homegear's RPC reference](https://ref.homegear.eu).

All RPC methods are callable on the Homegear object, i. e.:

```python
devices = hg.listDevices()
```

## Sending messages

To send a message, call the method `nodeOutput` on the Homegear object:

```python
msg = {"payload": "A message from Python."}
hg.nodeOutput(0, msg)
```

The first parameter is the index of the output, the second parameter is the message to send. Note that `msg` must be an `Array` and requires at least the entry `payload`.

If the node is sending a message in response to having received one, it should reuse the received message rather than create a new message object. This ensures existing properties on the message are preserved for the rest of the flow.

## Setting status

Whilst running, a node is able to share status information with the editor UI. This is done by calling the `nodeEvent` function:

```python
hg.nodeEvent('status/' + nodeId, {"text": "Feeling great", "shape": "dot", "fill": "green"})
```

## Using asyncio for thread synchronization

[`asyncio`](https://docs.python.org/3/library/asyncio.html) probably is the easiest way to synchronize threads. It's also the recommended way to implement asynchronous tasks in Python so it is used by many libraries.

Here's a full example where you don't have to worry about thread synchronization anymore:

```python
from homegear import Homegear
import threading
import sys
import asyncio

socketPath = sys.argv[1]
nodeId = sys.argv[2]
loop = asyncio.get_event_loop()
nodeInfo = None
hg = None
startUpComplete = threading.Condition()

# This callback method is called on Homegear variable changes.
def eventHandler(eventSource, peerId, channel, variableName, value):	
	# When the flows are fully started, "startUpComplete" is set to "true". Wait for this event.
	if eventSource == "nodeBlue" and peerId == 0 and variableName == "startUpComplete":
		startUpComplete.acquire()
		global nodeInfo
		nodeInfo = value
		startUpComplete.notify()
		startUpComplete.release()
    # Just call eventHandlerThreadSafe and do everything else there
    asyncio.run_coroutine_threadsafe(eventHandlerThreadSafe(eventSource, peerId, channel, variableName, value), loop).result()

async def eventHandlerThreadSafe(eventSource, peerId, channel, variableName, value):
    # Here we don't need to worry about thread synchronization anymore
    # Called for every subscribed Homegear variable event
    pass

# This callback method is called when a message arrives on one of the node's inputs.
def nodeInput(nodeInfo, inputIndex, message):
	# Just call nodeInputThreadSafe and do everything else there
	asyncio.run_coroutine_threadsafe(nodeInputThreadSafe(nodeInfo, inputIndex, message), loop).result()

async def nodeInputThreadSafe(nodeInfo, inputIndex, message):
    # Here we don't need to worry about thread synchronization anymore
    # Called for every new arriving message
    pass

async def appstart(loop, hg):
	while hg.connected():
        # Here is your main loop. We are just sleeping in this example.
		await asyncio.sleep(1)
	return 0

# hg waits until the connection is established (but for a maximum of 2 seonds).
# The socket path is passed in sys.argv[1], the node's ID in sys.argv[2]
hg = Homegear(socketPath, eventHandler, nodeId], nodeInput);

# Wait for the flows to start.
startUpComplete.acquire()
while hg.connected():
	if startUpComplete.wait(1) == True:
		break
startUpComplete.release()

# The node is now fully started. Start event loop.
loop.run_until_complete(appstart(loop, hg))
```

