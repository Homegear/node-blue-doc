# Node initialization and deinitialization

The initialization and deinitialization of nodes is a multi-step process. Not all functions are available in every step.

## Initialization

1. `init`: Right after the node is loaded init is called. A Struct with information about the node (id, name, configuration parameters, ...) is passed to init. All initilialization that can be done at this point, should be done here. When initialization fails, `false` must be returned, which aborts the initialization of that node. Homegear waits for `init` to complete for all nodes before continuing with the next step.
2. `start`: In start there are no restrictions on what is allowed to do. Just note, that `start` mustn't hang (don't sleep or wait). As with `init` `false` can be returned to abort the initialization of that node.
3. Once `start` is finished for all nodes, Homegear calls `startUpComplete`. `startUpComplete` must not hang as well.

## Deinitialization

1. `stop`: The deinitialization is started by Homegear calling `stop`. In `stop` all RPC methods and all other nodes are still available. Stop must not hang. For threads that means, you should trigger the stopping of threads in `stop`, but don't join them here. Homegear waits for all calls to `stop` to finish before continuing.
2. `waitForStop`: This method is made for one purpose: Joining threads. The stopping of threads should be triggered within stop. Every single thread should be stopped after 1 second and with this construct, all threads are finished after about one second.

## Overview

See the following table for an overview on what is allowed or available in the initialization/deinitialization methods:

|                                             | `init` | `start` | `startUpComplete` | `stop` | `waitForStop` |
| ------------------------------------------- | ------ | ------- | ----------------- | ------ | ------------- |
| **Node information available**              | yes    | yes     | yes               | yes    | yes           |
| **RPC methods available**                   | no     | yes     | yes               | yes    | no            |
| **Get parameters from configuration nodes** | no     | yes     | yes               | yes    | no            |
| **Must return immediately**                 | yes    | yes     | yes               | yes    | no            |
| **Can abort loading of node**               | yes    | yes     | no                | no     | no            |

​    

See the following table on the availability of the initialization/deinitialization methods for different programming languages:

|                    | `init` | `start` | `startUpComplete` | `stop`  | `waitForStop` |
| ------------------ | ------ | ------- | ----------------- | ------- | ------------- |
| **PHP (simple)**   | no     | no      | no                | no      | no            |
| **PHP (stateful)** | yes    | yes     | yes               | yes     | yes           |
| **JavaScript**     | no[^1] | no      | no                | no[^2]  | no            |
| **Python**         | no     | yes     | yes               | yes     | no            |
| **C++**            | yes    | yes[^3] | yes               | yes[^4] | yes           |

[^1]: JavaScript nodes are started and available before `init` is called on nodes of other languages.
[^2]: JavaScript nodes are stopped after all nodes written in other languages are stopped.
[^3]: The Python script is started in `start`.
[^4]: Signal 15 is sent to the Python script in `stop`.