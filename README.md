# pakontrol

Pakontrol is a simple python utility/daemon for controlling PulseAudio devices (sink, sink_input, source, source_output) from MIDI control messages.  It is intended to be small, have few requirements, and be easy to use.  I've tested it on Ubuntu 24 using a [nanoKontrol2](https://www.korg.com/us/products/computergear/nanokontrol2/).

## Requirements

All requirements are fairly mature and should be available in most distributions.

* pulsectl
* mido
* rtmidi
* python-daemon

Using `sudo apt install python3-mido python3-pulsectl python3-rtmidi python3-daemon` on Ubuntu should work

## Usage

```
usage: pakontrol [-h] [-v] [-f CONFIG_FILE] [-d] [-i]

options:
  -h, --help            show this help message and exit
  -v, --verbose
  -f CONFIG_FILE, --config-file CONFIG_FILE
  -d, --daemon
  -i, --interactive     interactive configuration mode
```

## Configuration

Pakontrol looks for configuration in `$PWD/pakontrol.conf` and `$HOME/.config/pakontrol/pakontrol.conf` and is in
ConfigParser (.ini) format.  See the `pakontrol.conf.sample` for an example.

You can use interactive mode to create a new configuration using your MIDI device and a PulseAudio mixer like
`pavucontrol`.  Just run `./pakontrol -i` and follow the prompts.

### Default Section

* `midi_device` - Name of the midi device to open.  Performs a regex match
* `midi_output` - Set to True if you want feedback sent back to your device
* `log_path` - Path to log file, or stdout if not specified
* `log_level` - Logging level, one of `DEBUG`, `INFO`, `WARNING`, `ERROR`
* `pid_file` - Path to pid file when using `--daemon` mode
* `multiplier` - Float multiplied by to map to volumes above 100%.  Defaults to 1.0.

### Rules

Other sections define rules to match a MIDI message to an audio device.  MIDI messages can be repeated
to map to more than one PulseAudio device (eg: all music apps).  Audio devices can be repeated too (eg: mapping
to mute button and volume slider).  See `pakcontrol.conf.sample` for examples.

* `midi_event` - Type of midi event, usually `control_change`
* `midi_channel` - Channel number
* `midi_control` - Control number
* `pa_type` - PulseAudio device type, one of `sink`, `sink_input`, `source`, `source_input`
* `pa_match_name` - Name of device to regex match on (optional)
* `pa_match_prop_named` - Name of property to match on (optional)
* `pa_match_prop_value` - Value of property to regex match on (optional)
* `pa_change` - Property to change/watch, one of `volume`, `mute`
* `multiplier` - Float multiplied by to map to volumes above 100%.  Defaults to 1.0.

Rules must specify `pa_match_name` and/or `pa_match_prop_named`/`pa_match_prop_value`.  Some devices have
names that change or are ambiguous, so the property match allows more flexibility.  For exmaple Rhythmbox
changes its' name depending on what's playing, so you can match on the property `application.name`.

