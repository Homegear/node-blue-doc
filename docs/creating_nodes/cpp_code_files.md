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

