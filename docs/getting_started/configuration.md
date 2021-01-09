# Configuration

Node-BLUE is configured in Homegear's `main.conf`. By default this file is located in `/etc/homegear`. The default settings should be fine for most installations. The following Node-BLUE-specific configuration options are available:

| Option                | Description                                                  |
| --------------------- | ------------------------------------------------------------ |
| `enableNodeBlue`      | Set to `true` to enable Node-BLUE.                           |
| `nodeBluePath`        | The path to Node-BLUE's static data. Shouldn't be changed when Homegear was installed using a package manager. |
| `nodeBlueDataPath`    | The path to Node-BLUE's dynamic data. This path can safely be changed. |
| `nodeBlueDebugOutput` | Enabled debug information in the UI (node highlighting, output values, ...). |
| `nodeBlueEventLimit1` | The limit of node highlighted nodes per tab per 10 seconds.  |
| `nodeBlueEventLimit2` | The limit for "last outputs" per tab per 10 seconds.         |
| `nodeBlueEventLimit3` | The limit for all other events (including debug tab and node outputs) per tab per 10 seconds. |

