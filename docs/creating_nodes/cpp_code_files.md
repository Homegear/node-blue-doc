# Code files of C++ nodes

C++ nodes are shared object files and must end on `.so`.

## Node class

The node is a C++ class derived from `Flows::INode` of the library `libhomegear-node`.

In addition a factory class derived from `Flows::NodeFactory` and a function `getFactory` are required.

`getFactory` must return an instance to the factory class.  The factory object must return a pointer to an instance of the node class.

This is required for Node-BLUE to be able to dynamically load the binary node file. As the factory part is the same for all nodes, you can just copy and paste it without having to worry about the inner workings.

### :fa-file: MyNode.h

```c++
#ifndef MY_NODE_H_
#define MY_NODE_H_

#include <homegear-node/NodeFactory.h>
#include <homegear-node/INode.h>

class MyFactory : Flows::NodeFactory {
 public:
  Flows::INode *createNode(const std::string &path, const std::string &type, const std::atomic_bool *frontendConnected) override;
};

extern "C" Flows::NodeFactory *getFactory();

namespace MyNode {

class MyNode : public Flows::INode {
 public:
  MyNode(const std::string &path, const std::string &type, const std::atomic_bool *frontendConnected);
  ~MyNode() override;
};

}

#endif
```

### :fa-file: MyNode.cpp

```c++
#include "MyNode.h"

Flows::INode *MyFactory::createNode(const std::string &path, const std::string &type, const std::atomic_bool *frontendConnected) {
  return new MyNode::MyNode(path, type, frontendConnected);
}

Flows::NodeFactory *getFactory() {
  return (Flows::NodeFactory *)(new MyFactory);
}

namespace MyNode {

MyNode::MyNode(const std::string &path, const std::string &type, const std::atomic_bool *frontendConnected) : Flows::INode(path, type, frontendConnected) {
}

MyNode::~MyNode() = default;

}
```

## Receiving messages

To receive messages, you need to implement the method `input`. This method is executed whenever a new message arrives.

### :fa-file: MyNode.h

```c++
void input(const Flows::PNodeInfo &info, uint32_t inputIndex, const Flows::PVariable &message) override;
```



### :fa-file: MyNode.cpp

```c++
void MyNode::input(const Flows::PNodeInfo &info, uint32_t inputIndex, const Flows::PVariable &message) {
  try {
    // Do something with 'message'
  }
  catch (const std::exception &ex) {
    _out->printEx(__FILE__, __LINE__, __PRETTY_FUNCTION__, ex.what());
  }
}
```

### Multiple inputs

If the node has more than one input, the input the message arrived at can be read from `inputIndex`.

## Handling errors and logging events

`Flows::INode` provides the `_out` object for logging messages. It provides the following methods. All methods have one parameter: The message string.

| Method          | Description                                                  |
| --------------- | ------------------------------------------------------------ |
| `printEx`       | Use this method to log exceptions with file name, line number and function name. |
| `printCritical` | Critical errors (log level 1)                                |
| `printError`    | Errors (log level 2)                                         |
| `printWarning`  | Warnings (log level 3)                                       |
| `printInfo`     | Info (log level 4)                                           |
| `printDebug`    | Debug (log level 5)                                          |
| `printMessage`  | Takes two parameters: Message and log level of the message.  |

The log message is written to Homegear's flows log file. Warnings and errors also trigger `Catch` nodes and - if not handled by a `Catch` node - are written to debug tabs of connected frontends.

## Calling Homegear RPC methods

Homegear RPC methods can be called with `invoke`:

```c++
auto parameters = std::make_shared<Flows::Array>();
parameters->reserve(2);
parameters->emplace_back(std::make_shared<Flows::Variable>("TEST"));
parameters->emplace_back(std::make_shared<Flows::Variable>("value"));
auto result = invoke("setSystemVariable", parameters);
```

For an overview of the available RPC methods, visit [Homegear's RPC reference](https://ref.homegear.eu).

## Sending messages

To send a message, just call the method `output` from any method of your class:

```c++
Flows::PVariable message = std::make_shared<Flows::Variable>(Flows::VariableType::tStruct);
message->structValue->emplace("payload", std::make_shared<Flows::Variable>("Using C++ to create nodes is even more awesome."));
output(0, message);
```

The first parameter is the index of the output, the second parameter is the message to send. Note that `message` must be a `Struct` and requires at least the entry `payload`.

If the node is sending a message in response to having received one,  it  should reuse the received message rather than create a new message object. This ensures existing properties on the message are preserved for the rest of the flow.

## Setting status

Whilst running, a node is able to share status information with the editor UI. This is done by calling the `nodeEvent` function:

```c++
Flows::PVariable status = std::make_shared<Flows::Variable>(Flows::VariableType::tStruct);
status->structValue->emplace("text", std::make_shared<Flows::Variable>("connected"));
status->structValue->emplace("fill", std::make_shared<Flows::Variable>("green"));
status->structValue->emplace("shape", std::make_shared<Flows::Variable>("dot"));
nodeEvent("status/" + _id, status);
```

## Flows::Variable

Values in nodes are represented by the type `Flows::Variable`. Any RPC function call and most methods in `Flows::INode` use this type. It is able to represent every data type required for RPC or required to communicate with code written in other programming languages. The supported types are listed in the enumeration `Flows::VariableType`:

```
enum class VariableType {
  tVoid = 0x00,
  tInteger = 0x01,
  tBoolean = 0x02,
  tString = 0x03,
  tFloat = 0x04,
  tArray = 0x100,
  tStruct = 0x101,
  //rpcDate = 0x10,
  tBase64 = 0x11,
  tBinary = 0xD0,
  tInteger64 = 0xD1,
  tVariant = 0x1111,
};
```

The constructors of `Flows::Variable` accept most primary data types. The value can be accessed by using:

* `stringValue`
* `integerValue`
* `integerValue64`
* `floatValue`
* `booleanValue`
* `arrayValue`
* `structValue`
* `binaryValue`

`Flows::Variable` is almost always used as a shared pointer. The shared pointer is represented by `Flows::PVariable`.

### Error Structs

Error Structs are Structs with the two properties `faultCode` of type `integer` and `faultString`. In addition the flag `errorStruct` must be set. The static method `PVariable createError(int32_t faultCode, std::string faultString)` can be used to easily construct an error Struct.

### Comparisons

All comparison operators are implemented for `Flows::Variable`. It can also be evaluated as a boolean value. The latter basically checks if it contains a non-empty, non-zero value.

### Copying

`Flows::Variable` is fully copyable.

### Printing

You can call `print` to get a String representation of the value.

## Flows::NodeInfo

An object of type `Flows::NodeInfo` is passed to `Flows::INode::init` and `Flows::INode::input` . This contains the following information:

| Variable                                  | Description                                                  |
| ----------------------------------------- | ------------------------------------------------------------ |
| `std::string id`                          | The node's ID                                                |
| `std::string flowId`                      | The flow's ID                                                |
| `std::string type`                        | The type of the node                                         |
| `Flows::PVariable info`                   | The full JSON of the node configuration including all settings, wires, etc. |
| `std::vector<std::vector<Wire>> wiresIn`  | All incoming wires. The outer vector are the inputs, the inner vector the wires to that input. |
| `std::vector<std::vector<Wire>> wiresOut` | All outgoing wires. The outer vector are the outputs, the inner vector the wires from that output. |

## Flows::MessageProperty

This class helps you working with inputs where a user can enter a message property (for example in the switch node). For example:

![image-20210123235734479](images/cpp_code_files/image-20210123235734479.png)

You can now construct the message property class with this setting:

```c++
auto property = Flows::MessageProperty("payload[5].data");
```

Now when a message arrives, you can extract that property:

```c++
auto myData = property.match(message);
//myData now contains the value of "payload[5].data"
```

In addition `Flows::MessageProperty` provides the following methods:

| Method                                                       | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `bool empty();`                                              | Just returns if a non-empty property was passed to the constructor. |
| `bool erase(Flows::PVariable &message);`                     | Deletes the property specified in the constructor from `message`. |
| `bool set(Flows::PVariable &message, Flows::PVariable &value);` | Sets the property in `message` to `value`.                   |

## Flows::JsonEncoder

Converts `Flows::PVariable` to a JSON string (`std::string Flows::JsonEncoder::getString(const PVariable &variable)`) or a JSON character vector (`std::vector<char> Flows::JsonEncoder::getVector(const PVariable &variable)`). Both methods can be accessed statically.

## Flows::JsonDecoder

Converts a JSON string (`PVariable decode(const std::string &json)`) or character vector (`PVariable decode(const std::vector<char> &json)`) to `Flows::PVariable`. Both methods can be accessed statically.

## Flows::Math

The `Flows::Math` class implements methods

* to convert strings to numbers
* scale or clamp numbers
* convert floating point numbers to and from binary and
* to convert floating point numbers to strings with defined precision.

Look at the header file in `libhomegear-node` for a description of the available methods.

## Flows::HelperFunctions

The `Flows::HelperFunctions` class implements methods:

* to trim Strings
* convert Strings to upper or lower case
* get the hexadecimal string representation of binary data
* get the current time in seconds, milliseconds, microseconds or a date and time String

For a description of the available methods see the header file in `libhomegear-node`.

## Flows::INode

!!! warning
    Please note, that no methods except `waitForStup` are allowed to block. Too many blocking calls would cause the Node-BLUE process to hang. There is no hard time limit, but blocking for 1 second already is too long. Consider moving long-running operations to a seperate thread.

### Variables

The following variables can be accessed from within you node class:

| Variable                                     | Description                                                  |
| -------------------------------------------- | ------------------------------------------------------------ |
| `std::string _path`                          | The full path to the node's files.                           |
| `std::string _type`                          | The node type.                                               |
| `std::string _id`                            | The node's ID.                                               |
| `std::string _flowId`                        | The node's flow ID.                                          |
| `std::string _name`                          | The name of the node (if set)                                |
| `const std::atomic_bool *_frontendConnected` | `true` when a Web browser with an open Node-BLUE editor is currently connected to Homegear. |
| `std::map<...> _localRpcMethods`             | A map to define RPC methods for inter-node communication. Used primarily to communicate with [configuration nodes](configuration_nodes.md). |

### Callable methods

TODO

### Overridable methods

#### Methods used for initialization and deinitialization

See section [initialization and deinitialization](node_initialization_and_deinitialization.md) for more information.

| Method                                  |
| --------------------------------------- |
| `bool init(const PNodeInfo &nodeInfo);` |
| `bool start();`                         |
| `void configNodesStarted();`            |
| `void startUpComplete();`               |
| `void stop();`                          |
| `void waitForStop();`                   |

#### Event methods

##### variableEvent

```c++
void variableEvent(const std::string &source,
                   uint64_t peerId,
                   int32_t channel,
                   const std::string &variable,
                   const PVariable &value,
                   const PVariable &metadata)
```

This method is called on any variable update to system variables, metadata variables and subscribed peer variables. To get peer variable updates, you need to subscribe them by calling `subscribePeer(uint64_t peerId, int32_t channel, const std::string &variable)`. As `subscribePeer` doesn't require inter-process or inter-node communication, it already can be called within `init`.

!!! note
    System variables and metadata variables don't require subscription.

###### Parameters

| Parameter  | Description                                                  |
| ---------- | ------------------------------------------------------------ |
| `source`   | The entity or service where the event originated: `device-<peer ID>`, `scriptEngine`, `profileManager`, `nodeBlue`, `ipcServer`, `homegear`, `client-<ID>`, `rpc-client-<ID>` or `mqtt` |
| `peerId`   | The ID of the peer the variable changed for. `0` for system variables |
| `channel`  | The channel of the peer the variable changed for. `-1` for metadata or system variables |
| `variable` | The name of the changed variable                             |
| `value`    | The new value                                                |
| `metadata` | Name of the peer; room, category and role information        |

##### flowVariableEvent

```c++
void flowVariableEvent(const std::string &flowId, const std::string &variable, const PVariable &value)
```

`flowVariableEvent` is called on any flow variable update within the flow the notified node is in. To receive flow variable events, you need to subscribe them calling `subscribeFlow()` (can already be called within `init`).

###### Parameters

| Parameter  | Description                                                  |
| ---------- | ------------------------------------------------------------ |
| `flowId`   | The ID of the flow. As the flow ID already is known this is redundant information |
| `variable` | The name of the flow variable                                |
| `value`    | The new value                                                |

##### globalVariableEvent

```c++
void globalVariableEvent(const std::string &variable, const PVariable &value)
```

`globalVariableEvent` is called on any Node-BLUE global variable update. To receive global variable events, you need to subscribe them calling `subscribeGlobal()` (can already be called within `init`).

###### Parameters

| Parameter  | Description                     |
| ---------- | ------------------------------- |
| `variable` | The name of the global variable |
| `value`    | The new value                   |

##### homegearEvent

```c++
void homegearEvent(const std::string &type, const PArray &data)
```

This is a "catch all" event handler. Every event handled by the Node-BLUE process triggers a call to this method. Use this method only when really required. To receive these events, you need to subscribe them by calling `subscribeHomegearEvents()`.

###### Parameters

| Parameter | Description                                                  |
| --------- | ------------------------------------------------------------ |
| `type`    | The type of event: `deviceVariableEvent`, `metadataVariableEvent`, `systemVariableEvent`, `flowVariableEvent`, `globalVariableEvent`, `variableProfileStateChanged`, `uiNotificationCreated`, `uiNotificationRemoved`, `uiNotificationAction`, `newDevices`, `deleteDevices` or `updateDevice`. |
| `data`    | The event-specific data.                                     |

##### statusEvent

```c++
void statusEvent(const std::string &nodeId, const PVariable &status)
```

When a node [sets it's status](node_status.md) a status event is triggered and `statusEvent` is called for all subscribed nodes. You can subscribe for status events by calling `subscribeStatusEvents()` (can already be called within `init`).

###### Parameters

| Parameter | Description                                                  |
| --------- | ------------------------------------------------------------ |
| `nodeId`  | The ID of the node the status is updated for (not the node triggering the status update). |
| `status`  | The [status object](node_status.md#status-object) |

##### errorEvent

```c++
bool errorEvent(const std::string &nodeId, int32_t level, const PVariable &error)
```

The method `errorEvent` is called for all subscribed nodes whenever a node logs an error or warning. You can subscribe for error events by calling `subscribeErrorEvents(bool catchConfigurationNodeErrors, bool hasScope, bool ignoreCaught)` (can already be called within `init`).

###### Parameters

| Parameter | Description                                                  |
| --------- | ------------------------------------------------------------ |
| `nodeId`  | The ID of the node logging the error                         |
| `level`   | `10` for critical, `20` for error, `30` for warning (Homegear's log level times 10) |
| `error`   | The error object containing information about the node and the error message |

