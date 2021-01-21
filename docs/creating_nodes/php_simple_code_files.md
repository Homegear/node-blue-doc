# Code files of simple PHP nodes

Simple PHP nodes are executed only when a message arrives. In contrast to stateful PHP nodes, they don't run continuously. A message arrives, the script is executed and then the script exits.

As the name suggests, this is probably the easiest possible way to create a node.

The file extension of simple PHP code files is `.php`.

## Node class

The node is a PHP class named `HomegearNode` derived from `HomegearNodeBase`. You must use the class name `HomegearNode` as otherwise Node-BLUE is not able to load the node.

```php
declare(strict_types=1);

class HomegearNode extends HomegearNodeBase
{
    ...
}
```

## Receiving messages

For every received message a new `HomegearNode` object is constructed by Node-BLUE. Only one method must exist for simple PHP nodes: `input`. This method is executed whenever a new message arrives.

```php
public function input(array $nodeInfo, int $inputIndex, array $msg)
{
	// Do something with '$msg'
}
```

### Multiple inputs

If the node has more than one input, the input the message arrived at can be read from `$inputIndex`.

## Handling errors and logging events

If the node encounters an error whilst handling the message or if it wants to write something to the log file, it should call the `log` method which is provided by `HomegearNodeBase`:

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

If the node is sending a message in response to having received one, it  should reuse the received message rather than create a new message  object. This ensures existing properties on the message are preserved  for the rest of the flow.

## Setting status

Whilst running, a node is able to share status information with the editor UI. This is done by calling the `nodeEvent` function:

```php
$this->nodeEvent('status/'.$nodeId, ['text' => 'Feeling great', 'shape' => 'dot', 'fill' => 'green']);
```

