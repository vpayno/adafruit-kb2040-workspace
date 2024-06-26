# Adafruit KB2040 TinyGo experiments

Journaling [Adafruit KB2040 TinyGo](https://tinygo.org/docs/reference/microcontrollers/feather-rp2040/) experiments here.

Hopefully This will work with the non-feather version of the board.

Hopefully TinyGo will work with the [Adafruit KB2040](https://www.adafruit.com/product/5302) since the documentation says
it works with the [Adafruit Feather KB2040](https://www.adafruit.com/product/4884).

## Links

- [Adafruit KB2040 Overview](https://learn.adafruit.com/adafruit-kb2040)
- [Adafruit KB2040 Manual](https://cdn-learn.adafruit.com/downloads/pdf/adafruit-kb2040.pdf)
- [Adafruit KB2040 Pinouts](https://learn.adafruit.com/adafruit-kb2040/pinouts)
- [TinyGo Tutorial](https://tinygo.org/docs/tutorials/)
- [TinyGo Documentation](https://tinygo.org/docs/)
- [TinyGo Drivers](https://github.com/tinygo-org/drivers)

## Installing Tools

Install TinyGo depdendencies:

```bash { background=false category=setup-tinygo closeTerminalOnSuccess=true excludeFromRunAll=true interactive=true interpreter=bash name=tinygo-install-dependencies promptEnv=true terminalRows=10 }
# install Atmel AVR microcontroller packages
sudo nala install -y --no-autoremove avr-libc avra avrdude avrdude-doc avrp dfu-programmer
```

Install TinyGo:

```bash { background=false category=setup-tinygo closeTerminalOnSuccess=true excludeFromRunAll=true interactive=true interpreter=bash name=tinygo-install-cli promptEnv=true terminalRows=10 }
# install tinygo deb package

set -e

wget -c "$(curl -sSL https://api.github.com/repos/tinygo-org/tinygo/releases/latest | jq -r '.assets[].browser_download_url' | grep amd64[.]deb)"
printf "\n"

sudo dpkg -i "$(curl -sSL https://api.github.com/repos/tinygo-org/tinygo/releases/latest | jq -r '.assets[].name' | grep 'amd64[.]deb')"
printf "\n"

tinygo version
printf "\n"
```

Setup new TinyGo module:

```bash { background=false category=setup-tinygo closeTerminalOnSuccess=true excludeFromRunAll=true interactive=true interpreter=bash name=tinygo-module-new promptEnv=true terminalRows=10 }
set -e

export PN="projectname"

mkdir "${PN}"
cd "${PN}"
go mod init "${PN}"
go get tinygo.org/x/drivers
```

## TinyGo CLI commands

Compile and export the the elf and uf2 files to the project directory.

Using target `feather-rp2040` seems to work for the Adafruit KB2040.

```bash { background=false category=build-tinygo closeTerminalOnSuccess=true excludeFromRunAll=true interactive=true interpreter=bash name=tinygo-cli-compile promptEnv=true terminalRows=25 }
# choose an tinygo project and build it

set -e

# all paths are relative to the /tinygo directory

stty cols 80
stty rows 25

declare PF
declare WD
declare FN

gum format "# Please choose a TinyGo project to build:"
printf "\n"
PF="$(gum choose $(find ./* -maxdepth 1 -type f -name '*[.]go';))"
printf "\n"

# working directory
WD="$(dirname "${PF}")"
cd "${WD}" || exit

# project file
FN="$(basename "${PF}")"

echo Running: rm -fv "${FN//.go/.elf}" "${FN//.go/.uf2}"
time rm -fv "${FN//.go/.elf}" "${FN//.go/.uf2}"
printf "\n"

echo Running: tinygo build -target=feather-rp2040 "${FN}"
time tinygo build -target=feather-rp2040 "${FN}"
printf "\n"

echo Running: elf2uf2-rs "${FN//.go/.elf}" "${FN//.go/.uf2}"
time elf2uf2-rs "${FN//.go/.elf}" "${FN//.go/.uf2}"
printf "\n"

file ./*elf ./*uf2
printf "\n"

ls -lhv ./*elf ./*uf2
printf "\n"
```

Before you can update the board, you need to reboot the Adafruit KB2040 into update mode by

- holding the `Boot` button
- pressing the `Reset` button
- let go of the `Boot` button
- wait for the `RPI-RP2` drive to show up

```bash { background=false category=deploy-tinygo closeTerminalOnSuccess=true excludeFromRunAll=true interactive=true interpreter=bash name=tinygo-cli-upload promptEnv=true terminalRows=25 }
# choose a tinygo project and deploy it

if [[ ! -d /mnt/chromeos/removable/RPI-RP2/ ]]; then
    printf "ERROR: You need to share the RPI-RP2 volume with Linux\n"
    exit 1
fi

if [[ ! -f /mnt/chromeos/removable/RPI-RP2/INFO_UF2.TXT ]]; then
    printf "ERROR: Board isn't in UF2 update mode\n"
    exit 1
fi

set -e

# all paths are relative to the /tinygo directory

stty cols 80
stty rows 25

declare PF
declare TD

gum format "# Please choose a TinyGo project to deploy:"
printf "\n"
PF="$(gum choose $(find . -type f -name '*.uf2'))"
printf "\n"

if [[ -z ${PF} ]]; then
    printf "ERROR: no UF2 file selected\n"
    exit 1
fi

gum format "# Please choose the deploy target directory:"
printf "\n"
TD="$(gum choose $(find /mnt/chromeos/removable/ -maxdepth 1 -type d | grep -v -E '^/mnt/chromeos/removable/$'))"
printf "\n"

echo Running: cp -v "${PF}" "${TD}"
cp -v "${PF}" "${TD}"
echo done.
```

## Experiments

### [Hello World](helloworld/)

Using the official [TinyGo Tutorial](https://tinygo.org/docs/tutorials/) for this experiment.

Ok, after a lot of searching for how to use a NeoPixel with TinyGo, got it to work.

- Serial output doesn't work.
- Using this [pinout](https://learn.adafruit.com/adafruit-kb2040/pinouts) guide.

Current LSP setup, linting, etc, not working very well with TinyGo.
I got most of my debugging help from `tinygo build`.
Still, Arduino and TinyGo seems like the best options for me so far.
