# 8]axe

The 8]axe is a 8-ASIC-BM1366 Miner based on the [0xaxe](https://github.com/shufps/0xaxe)

<img src="https://github.com/Patsch91/8Iaxe/blob/main/Platine.png" width="600px">


**rev1**: averaging at about 3.3-3,6TH/s with ~90 Watts on the wall on default settings (485Mhz) </br>
**rev2**: added boost converter to eliminate the extra 12V PSU / different design to fit vertain enclosures


**note**: The 8]axe is not a stand-alone device because it only supports USB. It can be run connected to a Raspberry Pi without problems. Also multiple devices can be connected to a single Pi. 

**WARNING**: The 8]Axe working with Voltage Domains is the end level boss for Pros. It's hard to get it working properly. The main reason is the powering scheme that uses 4 different voltage domain. It's difficult to get it into balance properly and needs very good and uniform cooling. The device works but can be super frustrating until it really is mining as it should. 
Therefore there is some WIP to eliminate the voltage-domains and chaining 8 Asics in one Domain instead. 
**There will be no further development in near future on this project by myself, due to another project *without* voltage-domains**
```

ASICs
=====

The 8]axe uses 8 ASICs of type BM1366

![image](https://github.com/Patsch91/8Iaxe/blob/main/8Iaxe%20pcb.png)

Compilation (Bootloader or CMSIS-DAP)
======================================
Start WSL on your PC and go through the followig steps:

```bash
# install curl
sudo apt install curl

# install rust
curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh

# add to ~/.bash.rc (afterwards, opening a new terminal is needed)
echo 'source "$HOME/.cargo/env"' >> ~/.bashrc

# clone repository
git clone https://github.com/shufps/0xaxe

# clone submodules
cd 0xaxe
git submodule init
git submodule update

# add rust target for STM32L0 variants
rustup target add thumbv6m-none-eabi

# build firmware for L072
cd firmware/fw-L072KZ
./build.sh
```

## Using it on Windows running WSL

For windows the usbipd is needed to connect the device to WSL. Install it from here:

https://github.com/dorssel/usbipd-win/releases

Afterwards reboot your PC and open the `cmd` as Administrator.

Connect the USB to your PC and power on the 12V while pressing the BOOT Button to start the device in BOOT-Mode

Then:
```
C:\Windows\System32>usbipd list
Connected:
BUSID  VID:PID    DEVICE                                                        STATE
2-1    046d:c52b  Logitech USB Input Device, USB-Eingabegerät                   Not shared
3-1    0c76:161f  USB PnP Audio Device, USB-Eingabegerät                        Not shared
3-2    046d:c52b  Logitech USB Input Device, USB-Eingabegerät                   Not shared
3-3    046d:c548  Logitech USB Input Device, USB-Eingabegerät                   Not shared
4-1    8087:0aa7  Intel(R) Wireless Bluetooth(R)                                Not shared
4-2    2e8a:000c  Picoprobe CMSIS-DAP v2, Serielles USB-Gerät (COM3)            Attached
```

Note the `BUSID` for the STM32. Then

```
C:\Windows\System32> usbipd bind --busid 4-2
```

Afterwards start your WSL Linux

Then attach the device with:
```
C:\Windows\System32> usbipd attach --wsl --busid 4-2
```

When it is running you can check in your WSL-Linux if the probe is detected:
```
probe-rs list

The following debug probes were found:
[0]: Picoprobe CMSIS-DAP (VID: 2e8a, PID: 000c, Serial: E66138528337A532, CmsisDap)
```

Installation via USB Bootloader on board with `BOOT` button
===========================================================
The STM32L072CB variant has an integrated DFU Bootloader that starts when pressing the `BOOT` button during reset.

Afterwards the firmware can be flashed via `dfu-utils`:

```bash
# install cargo-binutils and llvm tools
cargo install cargo-binutils
rustup component add llvm-tools-preview

# create the firmware.bin
DEFMT_LOG=info cargo objcopy --release --bin qaxe -- -O binary 0xaxe.bin

# install dfu-utils
sudo apt-get install dfu-util

# after booting, list the devices
dfu-util --list

# flash the binary
dfu-util -a 0 -s 0x08000000:leave -D 0xaxe.bin
```

Config 
=============
Use the "config.yml.example" of the [Piaxe-Miner](https://github.com/shufps/piaxe-miner) and change it as follows:

```bash
debug_bm1366: false
verify_solo: false
miner: 0xaxe

# works only with public pool
#suggest_difficulty: 2048

0xaxe:
  name: 8]axe
  chips: 8
  fan_speed_1: 0.8
  asic_frequency: 480
  extranonce2_interval: 1.5
  serial_port_asic: "/dev/ttyACM0"
  serial_port_ctrl: "/dev/ttyACM1"

...
```


Mining Client
=============

![image](https://github.com/Patsch91/8Iaxe/blob/main/Dashboard.png)


Stratum Mining Client:<br>
https://github.com/shufps/piaxe-miner


Case
=============
<img src="https://github.com/Patsch91/8Iaxe/blob/main/case.jpg" width="500px">
<img src="https://github.com/Patsch91/8Iaxe/blob/main/case2.jpg" width="500px">

I used a 80x160x170 aluminum case like this one: [Amazon](https://www.amazon.de/Aluminium-Projektbox-Mattschwarzer-DIY-Leiterplattenanschluss-W%C3%A4rmeableitungsgeh%C3%A4use-Bedienen/dp/B0CGGSVM8F/ref=sr_1_3?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&crid=3NMRXLZY6ALUN&dib=eyJ2IjoiMSJ9.0MLZnspoRUt-5Sq-UXD83eHRPPewKvMh4lus8XXQpkgDJRroXYZdpFcvkPHZnSEZWGdyZPu25JsKw8C_Gw7j3U3yo7FGxp-UKuHRwNwGErpQnLGFtEGB8CJPhlUupGPSyBMCgfSQbhhh6rR9keuntOinXBiLTcwFkg9jAHzRan196B2bXwHWc_GQhccHhde_KeXf4lkzuWefDuMZS-GZUgLpMEkL2ri5-xO686sFxk3LH6hBDYjgFnlqzBNBpYp2-c1hxcX88ikJwDevu7zkgC_WFYcVderTFYBcHsyTKOc.TUGJK2RRMmqnw5Lux7s7qib6OQUQhYEMLp2vokTvp34&dib_tag=se&keywords=80x160x170+case&qid=1723150526&sprefix=80x160x170+case%2Caps%2C94&sr=8-3)

Added 2x80mm Noctua Fans to get it pretty silent 

Misc
====
If you like this project and want to support future work, feel free to donate to: (https://github.com/shufps/0xaxe)



