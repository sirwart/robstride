# Robstride Python SDK

This is a simple SDK for communicating with Robstride motors in Python over CAN.

It also includes a simple `robstride` CLI client for making ad-hoc calls to your motors.

# Installation

The simplest way to install is via `pip`:

```python3 -m pip install robstride```

# Basic usage

This library is built on top of `python-can` and requires you to create a `can.Bus` object first.
Once you've configured your can bus you can create a `robstride.Client`, which allows you to make
calls to motors that are on the bus.

```
import can
import robstride
import time

with can.Bus() as bus:
    rs_client = robstride.Client(bus)

    # First set the run mode to position
    rs_client.write(1, 'run_mode', robstride.RunMode.Position)

    # Then enable the motor
    rs_client.enable(1)

    # Next tell the motor to go to its zero position
    # Many calls including enable and write return the current state of the motor
    resp = rs_client.write(1, 'loc_ref', 0)
    print('starting position:', resp.angle)

    # Give is a few seconds to reach that position
    time.sleep(3)

    # Now read the position, this time through the read call.
    new_angle = rs_client.read(1, 'mech_pos')
    print('ending position:', new_angle)

    # Finally deactivate the motor
    rs_client.disable(1)
```

# Basic usage of the CLI

All the calls that are supported by the library are also exposed via the `robstride` CLI client. It's especially useful for
performing one-off tasks such as changing the motor ID, which default to 127.

```
# Update the motor ID 127 to 1
robstride update-id 127 1
```

Other commands include `enable`, `disable`, `read`, and `write`.

# Configuring your CAN bus

Setting up your CAN bus depends on how you're commuinicating with CAN. If you're using a Canable compatible USB-CAN device,
you can set up your can bus as follows:

```
# Configure the can0 device to use 1Mbps
sudo ip link set can0 type can bitrate 1000000 loopback off

# Enable can0
sudo ifconfig can0 up
```

The simplest way to use this library is to configure your CAN setup in the `~/.can` file, which if you're using the setup above looks like this:
```
[default]
interface = socketcan
channel = can0
```

You can also configure these manually when creating the `can.Bus` object in code.
For the `robstride` CLI client you can configure these values with the optional `--interface` and `--channel` CLI arguments.