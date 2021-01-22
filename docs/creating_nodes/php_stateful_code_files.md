# Code files of stateful PHP nodes

In contrast to simple PHP nodes, stateful PHP nodes are executed on Homegear start or after a deploy and run until Homegear stops or the flow is redeployed.

Stateful PHP node code files end on `.s.php`.

## Node constructor

The node is a PHP class named `HomegearNode` derived from `HomegearNodeBase`. You must use the class name `HomegearNode` as otherwise Node-BLUE is not able to load the node.

```php
declare(strict_types=1);

class HomegearNode extends HomegearNodeBase
{
    ...
}
```

Node-BLUE calls a few methods on initialization and deinitialization which must be implemented. For details on when they are called and what they are doing, see [Node initialization and deinitialization](node_initialization_and_deinitialization.md).

```php
declare(strict_types=1);

class HomegearNode extends HomegearNodeBase
{
    private $hg = null;
    private $nodeInfo = null;

    public function __construct() {
        $this->hg = new \Homegear\Homegear();
    }

    function __destruct() {}
    
    /**
     * Called right after the constructor. Must return "true" on success.
     * When "false" is returned, the node initialization is aborted.
     * Don't call RPC methods here.
     */
    public function init(array $nodeInfo) : bool {
        $this->nodeInfo = $nodeInfo;
        
        //Do something
        
        return true;
    }

    /**
     * Called after init() is finished for all nodes. Must return "true"
     * on success. When "false" is returned, the node initialization is
     * aborted. RPC methods can be called here.
     */
    public function start() : bool {
        //Do something
        
        return true;
    }

    /**
     * Called after start() is finished for all nodes.
     */
    public function configNodesStarted() {
        //Do something
    }

    /**
     * Called after configNodesStarted() is finished for all nodes.
     */
    public function startUpComplete() {
        //Do something
    }
    
    /**
     * Trigger stopping of resources here. stop() is not allowed to hang
     * though.
     */
    public function stop() {
        //Do something
    }

    /**
     * Wait for resources to be freed here. waitForStop() is allowed to hang
     * for a maximum of 30 seconds.
     */
    public function waitForStop() {
        //Do something
    }
}
```

## Class constants

There are two class constants available:

```php
HomagearNode::NODE_ID
```

and

```php
HomegearNode::FLOW_ID
```

## Receiving messages

To receive messages, you need to implement the method `input`. This method is executed whenever a new message arrives. In contrast to simple PHP nodes, the same object instance is used for every invoke of this method. This means you can store data in variables which is available on the next call of `input`.

```php
public function input(array $nodeInfo, int $inputIndex, array $msg)
{
    // Do something with '$msg'
}
```

### Multiple inputs

If the node has more than one input, the input the message arrived at can be read from `$inputIndex`.

## Handling errors and logging events

If the node encounters an error whilst handling the message or if it  wants to write something to the log file, it should call the `log` method which is provided by `HomegearNodeBase`:

```php
$logLevel = 2;
$errorMessage = "Something really bad happened";
$this->log($logLevel, $errorMessage);
```

Available log levels are:

| Loglevel | Description |
| -------- | ----------- |
| 1        | Critical    |
| 2        | Error       |
| 3        | Warning     |
| 4        | Info        |
| 5        | Debug       |

The log message is written to Homegear's flows log file. Warnings and errors also trigger `Catch` nodes and - if not handled by a `Catch` node - are written to debug tabs of connected frontends.

## Calling Homegear RPC methods

You can create an object of the class Homegear to call Homegear RPC methods:

```php
$hg = new \Homegear\Homegear();
```

For an overview of the available RPC methods, visit [Homegear's RPC reference](https://ref.homegear.eu).

All RPC methods are callable on the Homegear object, i. e.:

```php
$hg->listDevices();
```

## Sending messages

To send a message, just call the method `output` from any method of your class:

```php
$msg = ['payload' => 'Using PHP to create nodes is awesome.'];
$this->output(0, $msg);
```

The first parameter is the index of the output, the second parameter is the message to send. Note that `$msg` must be an `Array` and requires at least the entry `payload`.

If the node is sending a message in response to having received one,  it  should reuse the received message rather than create a new message   object. This ensures existing properties on the message are preserved   for the rest of the flow.

## Setting status

Whilst running, a node is able to share status information with the editor UI. This is done by calling the `nodeEvent` function:

```php
$this->nodeEvent('status/'.$nodeId, ['text' => 'Feeling great', 'shape' => 'dot', 'fill' => 'green']);
```

## Concurrent tasks

The PHP extension "parallel" is included in Homegear's PHP for concurrent task support. See https://www.php.net/manual/en/book.parallel for more information.

An example implementation can be found here: https://github.com/Homegear/node-blue-node-fritzbox-callmonitor/blob/master/fritzbox-callmonitor.s.php (thanks to @job).

