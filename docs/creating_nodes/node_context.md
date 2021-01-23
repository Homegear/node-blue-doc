# Node context

A node can store data in Node-BLUE. This data is written into Homegear's database so it is still available after a restart of Node-BLUE or Homegear.

There are three scopes of context available to a node:

- Node - only visible to the node that set the value
- Flow - visible to all nodes on the same flow (or tab in the editor)
- Global - visible to all nodes

!!! note
    Please note that the Function nodes provide predefined functions to access context. A custom node must access context data for itself.

!!! note
    Configuration nodes that are used by and shared by other nodes are by default global, unless otherwise specified by the user of the node. As such it cannot be assumed that they have access to a Flow context.

## PHP

Context is available in simple and stateful PHP nodes.

```php
class HomegearNode extends HomegearNodeBase
{    
    public function myMethod {
        ...
        $nodeData = $this->getNodeData('my-node-info');
        $flowData = $this->getFlowData('my-flow-info');
        $globalData = $this->getGlobalData('my-global-info');
        
        $this->setNodeData('my-node-info', 5);        
        $this->setFlowData('my-flow-info', 'my string');
        $this->setGlobalData('my-global-info', ['key' => 'value']);
        ...
    }
}
```

## JavaScript

```javascript
function MyNode(config) {
    RED.nodes.createNode(this, config);
    var nodeData = this.homegear.invoke("getNodeData", [node.id, "my-node-info"]);
    var flowData = this.homegear.invoke("getFlowData", [node.id, "my-flow-info"]);
    var globalData = this.homegear.invoke("getGlobalData", [node.id, "my-global-info"]);
    
    this.homegear.invoke("setNodeData", [node.id, "my-node-info", 5]);
    this.homegear.invoke("setFlowData", [node.id, "my-flow-info", "my string"]);
    this.homegear.invoke("setGlobalData", [node.id, "my-global-info", {"key": "value"}]);
}
```

In addition it is possible to use Node-RED's context object. This requires more resources through.

```javascript
// Access the node's context object
var nodeContext = this.context();
var flowContext = this.context().flow;
var globalContext = this.context().global;

var nodeData = nodeContext.get("my-node-info");
var flowData = flowContext.get("my-flow-info");
var globalData = globalContext.get("my-global-info");

nodeContext.set("my-node-info", 5);
flowContext.set("my-flow-info", "my string");
globalContext.set("my-global-info", {"key": "value"});
```



## Python

```python
hg = Homegear(sys.argv[1], eventHandler, sys.argv[2], nodeInput)

...

nodeData = hg.getNodeData("my-node-info")
flowData = hg.getFlowData("my-flow-info")
globalData = hg.getGlobalData("my-global-info")

hg.setNodeData("my-node-info", 5)
hg.setFlowData("my-flow-info", "my string")
hg.setFlowData("my-global-info", {"key": "value"})
```

## C++

```c++
auto nodeData = getNodeData("my-node-info");
auto flowData = getFlowData("my-flow-info");
auto globalData = getGlobalData("my-global-info");

setNodeData("my-node-info", std::make_shared<Flows::Variable>(5));
setFlowData("my-flow-info", std::make_shared<Flows::Variable>("my string"));
auto myStruct = std::make_shared<Flows::Variable>(Flows::VariableType::tStruct);
myStruct->emplace("key", std::make_shared<Flows::Variable>("value"));
setGlobalData("my-global-info", myStruct);
```

