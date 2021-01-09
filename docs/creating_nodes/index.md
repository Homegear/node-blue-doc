# Creating Nodes

Node-BLUE can be easily extended by creating new nodes. New nodes can be written in four programming languages:

1. C++
2. PHP
3. Python
4. JavaScript

This guide describes how to write new nodes in each of them. As the Node-BLUE frontend is based on Node-RED, this documentation is largely based on and citing the [Node-RED documentation](https://nodered.org/docs/creating-nodes/) as well.

Please click on the sections you want to read:

* [Creating your first node](first_node.md)
* [Node initialization and deinitialization](node_initialization_and_deinitialization.md)

## General guidance

When developing new nodes, please follow the following principles:

* Nodes should be simple to use, regardless how complex the underlying functionality is.
* When a node outputs data, the main information should always be encoded in `msg.payload`.
* It should never be required to use a function node in combination with nodes.
* Nodes should have a good documentation.
* Errors should always be catched, otherwise they might crash the whole process.

## License

Except as otherwise provided, content on this site, including all materials posted by the ib company GmbH or Homegear GmbH, is licensed under a Creative Commons Attribution 4.0 License.