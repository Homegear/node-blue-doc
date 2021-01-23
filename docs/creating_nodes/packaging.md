# Packaging

This section is largely citing the Node-RED documentation.

Nodes can be packaged as modules. This makes them easy to install along with any dependencies they may have.

## Naming

If you wish to use **node-blue** in the name of your node please use `node-blue-node-` as a prefix to their name. Alternatively, any name that doesn’t use `node-blue` as a prefix can be used.

## Directory structure

Here is a typical directory structure for a node package:

```
├── LICENSE
├── package.json
├── README.md
└── sample
    ├── icons
    │   └── my-icon.svg
    ├── locales
    │   ├── de
    │   │   ├── sample.help.html
    │   │   └── sample.json
    │   └── en
    │       ├── sample.help.html
    │       └── sample.json
    ├── sample.cred.json
    ├── sample.html
    └── sample.php
```

There are no strict requirements over the directory structure used within the package. If a package contains multiple nodes, they could all exist in the same directory, or they could each be placed in their own sub-directory.

## package.json

Along with the usual entries, the `package.json` file must contain a `node-blue` entry that lists the code files that contain nodes for the runtime to load.

Nodes written in some programming languages also require an `maxThreadCounts` entry. Just set this entry to `0` when you don't start any threads.

!!! note
    Please do NOT add the `node-blue` keyword until you are happy that the node is stable and working correctly, and documented sufficiently for others to be able to use it.

```json
{
    "name"         : "node-blue-node-samplenode",
    "version"      : "0.0.1",
    "description"  : "A sample node for node-blue",
    "dependencies": {
    },
    "keywords": [ "node-blue" ],
    "node-blue": {
        "maxThreadCounts": {
            "sample": 1
        },
        "nodes": {
            "sample": "sample/sample.s.php"
        }
    }
}
```

## README.md

The README.md file should describe the capabilities of the node, and list any pre-requisites that are needed in order to make it function. It may also be useful to include any extra instructions not included in the *info* tab part of the node’s html file, and maybe even a small example flow demonstrating it’s use.

The file should be marked up using [GitHub flavoured markdown](https://help.github.com/articles/markdown-basics/).

## LICENSE

Please include a license file so that others may know what they can and cannot do with your code.

## Publishing

Just upload your node to any publicly available Git repository.

## Adding to Node-BLUE palette

Please post a submission request in the [Homegear forum](https://forum.homegear.eu). Before doing that, make sure all of the packaging requests are met.