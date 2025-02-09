← [Back](../README.md)

# 3. Configuration

## 3.1 Device Configuration

Device Configuration is stored in configuration object - `fair::mq::ProgOptions`. It is accessible by the device, plugins or from DeviceRunner/main:

Plugins <---read/write---> ProgOptions <---read/write---> Device

Whenever a configuration property is set, it is set in ProgOptions. Device/Channels/User code read this value and apply it as necessary at different stages:
 - apply it immidiately
 - apply it in device/channels during InitializingDevice/Binding/Connecting states

Here is an overview of the device/channel options and when they are applied:

| Property | Applied in |
| --- | --- |
| `severity` | immidiately (if `fair::mq::DeviceRunner` is used (also the case when using `<runFairMQDevice.h>`)) |
| `file-severity` | immidiately (if `fair::mq::DeviceRunner` is used (also the case when using `<runFairMQDevice.h>`)) |
| `verbosity` | immidiately (if `fair::mq::DeviceRunner` is used (also the case when using `<runFairMQDevice.h>`)) |
| `color` | immidiately (if `fair::mq::DeviceRunner` is used (also the case when using `<runFairMQDevice.h>`)) |
| `log-to-file` | immidiately (if `fair::mq::DeviceRunner` is used (also the case when using `<runFairMQDevice.h>`)) |
| `id` | at the end of `fair::mq::State::InitializingDevice` |
| `io-threads` | at the end of `fair::mq::State::InitializingDevice` |
| `transport` | at the end of `fair::mq::State::InitializingDevice` |
| `network-interface` | at the end of `fair::mq::State::InitializingDevice` |
| `init-timeout` | at the end of `fair::mq::State::InitializingDevice` |
| `shm-segment-size` | at the end of `fair::mq::State::InitializingDevice` |
| `shm-monitor` | at the end of `fair::mq::State::InitializingDevice` |
| `ofi-size-hint` | at the end of `fair::mq::State::InitializingDevice` |
| `rate` | at the end of `fair::mq::State::InitializingDevice` |
| `session` | at the end of `fair::mq::State::InitializingDevice` |
| `chan.*` | at the end of `fair::mq::State::InitializingDevice` (channel addresses can be also applied during `fair::mq::State::Binding`/`fair::mq::State::Connecting`) |

## 3.2 Configuration options

## 3.2 Communication Channels Configuration

The communication channels can be configured via configuration parsers. The parser system is extendable, so if provided parsers do not suit your style, you can write your own and plug them in the configuration system.

The provided parsers are:

### 3.2.1 JSON Parser

This parser reads channel configuration from a JSON file. Example:

```JSON
{
    "fairMQOptions": {
        "devices": [
            {
                "id": "sampler1",
                "channels": [
                    {
                        "name": "data",
                        "sockets": [
                            {
                                "type": "push",
                                "method": "bind",
                                "address": "tcp://*:5555",
                                "sndBufSize": 1000,
                                "rcvBufSize": 1000,
                                "sndKernelSize" : 0,
                                "rcvKernelSize" : 0,
                                "transport": "shmem",
                                "linger": "500",
                                "portRangeMin": "22000",
                                "portRangeMax": "23000",
                                "autoBind": false,
                                "numSockets": 0,
                                "rateLogging": 1
                            }
                        ]
                    }
                ]
            },
            {
                "id": "sink1",
                "channels": [
                    {
                        "name": "data",
                        "sockets": [
                            {
                                "type": "pull",
                                "method": "connect",
                                "address": "tcp://localhost:5555",
                                "transport": "shmem"
                            }
                        ]
                    }
                ]
            }
        ]
    }
}
```

The JSON file can contain configuration for multiple devices.

- The mapping between device and configuration happens via the device ID (the `--id` parameter of the launched device and the `"id"` entry in the config).
- Instead of `"id"`, JSON file may contain device configurations under `"key"`, which allows launched devices to share configuration, e.g.: `my-device-executable --id <device-id> --config-key <config-key>`.
- Socket options must contain at least *type*, *method* and *address*, the rest of the values are optional and will get default values of the channel.
- If a channel has multiple sub-channels, common properties can be defined under channel directly, and will be shared by all sub-channels, e.g.:

```JSON
"channels": [{
    "name": "data",
    "type": "push",
    "method": "bind",
    "sockets": [{
        "address": "tcp://*:5555",
        "address": "tcp://*:5556",
        "address": "tcp://*:5557"
    }]
}]
```

### 3.2.2 SuboptParser

This parser configures channels directly from the command line.
The parser handles a comma separated key=value list format by using the getsubopt function of the standard library.
The option key `--channel-config` can be used with the list of key/value pairs, e.g.:

```
--channel-config name=output,type=push,method=bind,address=tcp://127.0.0.1:5555
```

## 3.3 Introspection

A compiled device executable repots its available configuration. Run the device with one of the following options to see the corresponding help:

- `-h [ --help ]`: All available command line options with their descriptions and default values.

- `--print-options`: All available command line options in a machine-readable format: `<option>:<computed-value>:<<type>>:<description>`.

- `--print-channels`: Prints registered channels in a machine-readable format: `<channel name>:<minimum sub-channels>:<maximum sub-channels>`. There are devices where channels names are not known in advance before the configuration takes place (e.g. FairMQMultiplier has configurable channel names at runtime). This options will only print channels that have been registered in the device by implementing the following method:

```C++
void YourDevice::RegisterChannelEndpoints()
{
    // provide channel name, minimum and maximum number of subchannels
    RegisterChannelEndpoint("channelA", 1, 10000);
    RegisterChannelEndpoint("channelB", 1, 1);
}
```

← [Back](../README.md)
