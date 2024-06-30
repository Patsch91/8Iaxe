# 8]axe

The 8]axe is a 8-ASIC-BM1366 Miner based on the [0xaxe](https://github.com/shufps/0xaxe)

<img src="https://github.com/Patsch91/8Iaxe/blob/main/8Iaxe%20assembled.png" width="600px">


**rev1**: working at about 3.2-3,5TH/s with ~90 Watts on the wall</br>


**note**: The 8]axe is not a stand-alone device because it only supports USB. It can be run connected to a Raspberry Pi without problems. Also multiple devices can be connected to a single Pi. 

ASICs
=====

The 8]axe uses 8 ASICs of type BM1366

![image](https://github.com/Patsch91/8Iaxe/blob/main/8Iaxe%20pcb.png)

Compilation (Bootloader or CMSIS-DAP)
======================================

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

now start the stm32 in DFU mode by pressing `boot` 

# after booting, list the devices
dfu-util --list

# flash the binary
dfu-util -a 0 -s 0x08000000:leave -D 0xaxe.bin
```



Mining Client
=============

![image]


Stratum Mining Client:<br>
https://github.com/shufps/piaxe-miner

Misc
====
If you like this project and want to support future work, feel free to donate to: `bc1q29hp4fqtks2wzpmfwtpac64fnr8ujw2nvnra04`



