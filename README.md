# Prometheus Brickd Exporter

The brickd exporter is a Prometheus exporter which connects to a [Tinkerforge](https://www.tinkerforge.com/)
[brickd](https://www.tinkerforge.com/en/doc/Software/Brickd.html) and exports the values from the
connected bricks and bricklets.

Data from the brickd is collected in the background. Currently callbackPeriod is set to 10,000 ms, which can
be changed in the config.

Note: the `sub_id` label has been deprecated, it is replaced by the `sensor_id` label in the metrics and
will be removed in the future.


## Usage

### Pre-requisite

go version 1.11 or later (go modules are used in this repo).

### Building the brickd exporter

Clone the repository via `go get github.com/vetinari/brickd_exporter` or git and cd into the directory.
Then build the brickd\_exporter binary with the following commands.

    $ go build

### Configuration

When no `--config.file /path/to/brickd.yml` is given, the default config is:

```yaml
collector:
    log_level: info
    callback_period: 10s
    ignored_uids: []
    expire_period: 0s
listen:
    address: :9639
    metrics_path: /metrics
brickd:
    address: localhost:4223
```

Any of these values can be set. Use the default `brickd.address` when the bricks are connected
on local USB port.

`collector.log_level` can be set to `debug` to see the devices discovered and their values received
from the callbacks.

`collector.labels` is a key -> value map of strings which will be applied to all metrics.

`collector.sensor_labels` is a mapping of the UID of the brick(let), to sensor id (as string, usually
`"0"` for all except with the "Outdoor Weather Bricklet" to a key -> value map of strings, see [brickd.yml](brickd.yml)
for examples. Those will only applied to the defined sensors.

`collector.expire_period` sets a duration after which old values are not exported anymore, i.e. if the latest value of a 
brick / bricklet has been received from brickd more than this period ago it will not be shown anymore. `0s` (or any other
`time.Duration` of `0` disables this feature (the default). Do not set this too low or you might not export anything :) 
Depending on your use case 2 or more times the `collector.callback_period` should be OK.

### Running

Start with `--config.file /path/to/brickd.yml` to pass a config file. 

## Suported bricks and bricklets

Bricks:

* [Master Brick](https://www.tinkerforge.com/en/doc/Hardware/Bricks/Master_Brick.html)
* [Zero Hat Brick](https://www.tinkerforge.com/de/doc/Hardware/Bricks/HAT_Zero_Brick.html)

Bricklets:

* [AirQuality Bricklet](https://www.tinkerforge.com/en/doc/Hardware/Bricklets/Air_Quality.html)
* [Barometer Bricklet](https://www.tinkerforge.com/en/doc/Hardware/Bricklets/Barometer.html)
* [Barometer Bricklet v2.0](https://www.tinkerforge.com/en/doc/Hardware/Bricklets/Barometer_V2.html)
* [Humidity Bricklet](https://www.tinkerforge.com/en/doc/Hardware/Bricklets/Humidity.html)
* [Humidity Bricklet v2.0](https://www.tinkerforge.com/en/doc/Hardware/Bricklets/Humidity_V2.html)
* [Outdoor Weather Bricklet](https://www.tinkerforge.com/en/doc/Hardware/Bricklets/Outdoor_Weather.html)

Adding more is easy, see [Contributing](#contributing)

## Contributing

If you would like to contribute code or documentation, follow these steps:

* Clone a local copy.
* Make your changes on a uniquely named branch.
* Comment those changes.
* Test those changes 
* Make sure the code is go formatted (hint: `gofmt -w $file`)
* Push your branch to a fork and create a Pull Request.

### Adding new bricks and bricklets

* add register function, check the existing ones in collector/bricks.go / collector/bricklets.go
  as examples.
* add the functions to the `NewCollector`, example of existing ones:
```go
      brickd.Devices = map[uint16]RegisterFunc{
        master_brick.DeviceIdentifier:      brickd.RegisterMasterBrick,
        humidity_bricklet.DeviceIdentifier: brickd.RegisterHumidityBricklet,
      }
```
* don't forget to update the imports
* test new devices and create a pull request (see above).
