# Picroft with JBL Go

Picroft setup with support for JBL Go speakers.

## TODOs

- [ ] Improve speaker sound quality (specially for youtube skill)
- [ ] Use Raspbian image instead of Picroft's (maybe using 5GHz WiFi improves audio quality)
- [ ] Simplify setup with CustomPiOS

## Manual setup

### Requirements

* Raspberry Pi 3B, 3B+, or 4 with its power supply
* JBL Go bluetooth speakers (version 1 and 2 tested)
* microSD card (>8Gb) with adapter for your laptop 

### Steps

1. Download a Picroft image and burn it into your microSD card:
```shell
wget https://mycroft.ai/to/picroft-rc -O picroft.img.zip
unzip picroft.img.zip
dd bs=4M if=Picroft_v21.02.0_20210604.img of=/dev/mmcblk0 conv=fsync
```
Here we use the RC image because the `v20.08` image is not compatible with a
Raspberry Pi 4 8Gb.

2. Configure WiFi headless mode (*optional* - use ethernet alternatively): mount
the microSD's `boot` partition and create in it a file named
`wpa_supplicant.conf` with the following content:
```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=US
network={
        ssid="<wifi_ssid>"
        psk="<wifi_password>"
}
```
Note that the Picroft image is only compatible with 2.4GHz networks only (even
though Raspbian images support 5GHz).

3. Umount and extract the microSD card, plug it onto the Pi and turn it on.
Check your router's connected devices for the local IP for `picroft`.

4. SSH into your Pi. The default password if `mycroft`. Stop any Mycroft scripts
if necessary (CTRL+C).
```shell
ssh pi@<ip_address>
```

5. Install PulseAudio's bluetooth module
```shell
sudo apt-get update
sudo apt-get install pulseaudio-module-bluetooth
```
A reboot may be neccessary. Additionally, bluetooth's service errors on the Pi
can be fixed by following [these steps](https://peppe8o.com/fixed-connect-bluetooth-headphones-with-your-raspberry-pi/).

6. Turn on your JBL Go speakers, set them to pairing mode, and pair the device
with your Pi.
```shell
bluetoothctl
[bluetooth]# power on
[bluetooth]# agent on
[bluetooth]# default-agent
[bluetooth]# scan on
...
[NEW] Device XX:XX:XX:XX:XX:XX JBL GO
...
[bluetooth]# pair XX:XX:XX:XX:XX:XX
[bluetooth]# trust XX:XX:XX:XX:XX:XX
[bluetooth]# connect XX:XX:XX:XX:XX:XX
[JBL GO]# quit
```
We'll use `XX:XX:XX:XX:XX:XX` from here
on but replace it with your JBL Go's address.

7. Edit `/home/pi/custom_setup.sh` to automate bluetooth pairing and PulseAudio
setup. It should look like this:
```
#!/bin/bash
# Use this script to execute custom actions on startup.
printf "### Connect Bluetooth Speaker ###\n"
echo "power on" | bluetoothctl
echo "agent on" | bluetoothctl
echo "default-agent" | bluetoothctl
echo "connect XX:XX:XX:XX:XX:XX" | bluetoothctl
printf "### Configuring PulseAudio ###\n"
sleep 10
pacmd set-card-profile bluez_card.XX_XX_XX_XX_XX_XX headset_head_unit
pacmd set-default-sink bluez_sink.XX_XX_XX_XX_XX_XX.headset_head_unit
pacmd set-default-source bluez_source.XX_XX_XX_XX_XX_XX.headset_head_unit
printf "### Config Done! ###\n"
```
Note we use underscores instead of colons with the `pacmd` command. You can
check that your JBL Go is ready by running `pacmd list-sources | grep name:` and
`pacmd list-sinks | grep name:`.

8. Reboot, wait until your JBL Go is paired and run Mycroft's setup wizard:
```shell
mycroft-setup-wizard
```
Choose USB devices for both the speakers and the microphone, bluez should be
listed as options.

## Custom image

WIP. We'll use [CustomPiOS](https://github.com/guysoft/CustomPiOS) to generate
an image with all the setup done.

## Custom skills

### Play from youtube

```
sudo apt-get install mplayer
# sudo apt-get install ffmpeg
msm install https://github.com/hexeratops/mycroft-youtube-skill
```

## References

* https://community.mycroft.ai/t/bluetooth-mic-speaker-combo/3002
* https://affengriff.net/2021/10/25/picroft-jbl-go-speaker/
* https://peppe8o.com/fixed-connect-bluetooth-headphones-with-your-raspberry-pi/
* https://community.mycroft.ai/t/bluetooth-hsp-hfp/8199
* https://community.mycroft.ai/t/input-output-with-bluetooth-speaker/8353
