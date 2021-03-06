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
* [JavaScript code files](javascript_code_files.md)
* [Code files of simple PHP nodes](php_simple_code_files.md)
* [Code files of stateful PHP nodes](php_stateful_code_files.md)
* [Code files of Python nodes](python_code_files.md)
* [Code files of C++ nodes](cpp_code_files.md)
* [HTML file](html_file.md)
* [Packaging](packaging.md)
* [Node properties](node_properties.md)
* [Node credentials](node_credentials.md)
* [Node appearance](node_appearance.md)
* [Node edit dialog](node_edit_dialog.md)
* [Editor UI widgets](editor_ui_widgets.md)
* [Storing context](node_context.md)
* [Node status](node_status.md)
* [Configuration nodes](configuration_nodes.md)
* [Help style guilde](node_help_style_guide.md)
* [Adding examples](examples.md)
* [Internationalization](internationalization.md)

## General guidance

When developing new nodes, please follow the following principles:

* Nodes should be simple to use, regardless how complex the underlying functionality is.
* When a node outputs data, the main information should always be encoded in `msg.payload`.
* It should never be required to use a function node in combination with nodes.
* Nodes should have a good documentation.
* Errors should always be catched, otherwise they might crash the whole process.

## License

Except as otherwise provided, content on this site, including all materials posted by the ib company GmbH or Homegear GmbH, is licensed under a Creative Commons Attribution 4.0 License.