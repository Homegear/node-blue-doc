# JavaScript code files

This section is largely citing the Node-RED documentation.

For JavaScript nodes the node `.js` file defines the runtime behavior of the node.

!!! warning:
    Don't apply the documentation for JavaScript nodes to nodes written in other languages. A lot of things described in the JavaScript sections are valid for JavaScript nodes only.

## Node constructor

Nodes are defined by a constructor function that can be used to create new instances of the node. The function gets registered with the runtime so it can be called when nodes of the corresponding type are deployed in a flow.

The function is passed an object containing the properties set in the flow editor.

The first thing it must do is call the `RED.nodes.createNode` function to initialize the features shared by all nodes. After that, the node-specific code lives.

```javascript
function SampleNode(config) {
    RED.nodes.createNode(this,config);
    // node-specific code goes here

}

RED.nodes.registerType("sample",SampleNode);
```

## Receiving messages

Nodes register a listener on the `input` event to receive messages from the up-stream nodes in a flow.

```javascript
this.on('input', function(msg, send, done) {
    // do something with 'msg'

    // Once finished, call 'done'.
    // This call is wrapped in a check that 'done' exists
    // so the node will work in earlier versions of Node-RED (<1.0)
    if (done) {
        done();
    }
});
```

### Multiple inputs

If the node has more than one input, the input the message arrived at can be read from `msg.inputIndex`.

## Handling errors

If the node encounters an error whilst handling the message, it should pass the details of the error to the `done` function.

This will trigger any Catch nodes present on the same tab, allowing the user to build flows to handle the error.

```javascript
let node = this;
this.on('input', function(msg, send, done) {
    // do something with 'msg'

    // If an error is hit, report it to the runtime
    if (err) {
        done(err);
    }
});
```

## Sending messages

If the node sits at the start of the flow and produces messages in response to external events, it should use the `send` function on the Node object:

```javascript
var msg = { payload:"hi" }
this.send(msg);
```

If the node wants to send from inside the `input` event listener, in response to receiving a message, it should use the `send` function that is passed to the listener function:

```javascript
let node = this;
this.on('input', function(msg, send, done) {
    // For maximum backwards compatibility, check that send exists.
    // If this node is installed in Node-RED 0.x, it will need to
    // fallback to using `node.send`
    send = send || function() { node.send.apply(node,arguments) }

    msg.payload = "hi";
    send(msg);

    if (done) {
        done();
    }
});
```

If `msg` is null, no message is sent.

If the node is sending a message in response to having received one, it should reuse the received message rather than create a new message object. This ensures existing properties on the message are preserved for the rest of the flow.

### Multiple outputs

If the node has more than one output, an array of messages can be passed to `send`, with each one being sent to the corresponding output.

```javascript
this.send([ msg1 , msg2 ]);
```

### Multiple messages

It is possible to send multiple messages to a particular output by passing an array of messages within this array:

```javascript
this.send([ [msgA1 , msgA2 , msgA3] , msg2 ]);
```

## Closing the node

Whenever a new flow is deployed, the existing nodes are deleted. If any of them need to tidy up state when this happens, such as disconnecting from a remote system, they should register a listener on the `close` event.

```javascript
this.on('close', function() {
    // tidy up any state
});
```

If the node needs to do any asynchronous work to complete the tidy up, the registered listener should accept an argument which is a function to be called when all the work is complete.

```javascript
this.on('close', function(done) {
    doSomethingWithACallback(function() {
        done();
    });
});
```

If the registered listener accepts two arguments, the first will be a boolean flag that indicates whether the node is being closed because it has been removed entirely, or that it is just being restarted. It will also be set to `true` if the node has been disabled.

```javascript
this.on('close', function(removed, done) {
    if (removed) {
        // This node has been disabled/deleted
    } else {
        // This node is being restarted
    }
    done();
});
```

### Timeout behaviour

The runtime will timeout the node if it takes longer than 15 seconds. An error will be logged and the runtime will continue to operate.

## Logging events

If a node needs to log something to the Homegear log, it can use one of the following functions:

```javascript
this.log("Something happened");
this.warn("Something happened you should know about");
this.error("Oh no, something bad happened");
```

The `warn` and `error` messages also get sent to the flow editor debug tab.

## Setting status

Whilst running, a node is able to share status information with the editor UI. This is done by calling the `status` function:

```javascript
this.status({fill:"red",shape:"ring",text:"disconnected"});
```

The details of the status api can be found here (TODO).