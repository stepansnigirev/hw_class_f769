# Hardware wallet in micro-python on STM32F769-Discovery board

Currently focusing on [STM32F769DISCO](https://www.st.com/en/evaluation-tools/32f769idiscovery.html) developer board. Support of other boards comes as soon as this one is stable enough.

## Project structure

`micropython` folder contains a fork of [micropython](http://micropython.org) with some temporary workarounds for stm32f769 board. Currently the master branch of micropython fails to build for this target, as soon as it is fixed we will switch to the main micropython repo.

`usermods/bitcoin` folder contains minimal C-modules for micropython with basic crypto algorithms. Interface to optimized C modules allow to use optimized elliptic curve arithmetics and hashing functions:
- `usermods/bitcoin/hashlib.c` adds support of sha1, sha256, sha512 and ripemd160 required for Bitcoin to `hashlib` python module.
- `usermods/bitcoin/ecc.c` adds a few optimized elliptic curve operations (exposed to micropython as `_ecc` module).
- `usermods/display/` folder contains a C-module to work with the display present on the board.

`libs` folder contains python modules that makes it more convenient to work with optimized C libraries described above:
- `libs/bitcoin` defines classes useful for Bitcoin. `PrivateKey` and `PublicKey` classes support scalar and point arithmetics - they implement `__mul__`, `__add__` and other operator overrides that are possible for corresponding classes.
- `libs/hmac.py` and `libs/pbkdf2.py` do what their names say they do ;)

## Compiling

Preparation steps:

## To run on STM32F769DISCO board:

Preparation steps:

```
git clone https://github.com/stepansnigirev/hw_class_f769.git --recursive
cd micropython
make -C mpy-cross
```

Compilation for the board:

```
cd ports/stm32
make BOARD=STM32F769DISC USER_C_MODULES=../../../usermods
arm-none-eabi-objcopy -O binary build-STM32F769DISC/firmware.elf upy-f769disco.bin
```

Then copy `upy-f769disco.bin` to the board. When you connect the board with the second USB cable it will mount a `PYBFLASH` volume where you can put your python code.

When micropython is on the board copy content of the `lib` folder to the `PYBFLASH` drive - now you have access to all the libraries and can work on your hardware wallet.

# API

## `display` library

Works on ly on STM32F769DISCO right now, and only a few functions are currently supported:
- `init()` - inits the screen and clears it. Should be run first.
- `print(msg, x, y)` - prints the `msg` string at coordinates `(x,y)`. Doesn't support `\n`.
- `clear()` - clears the screen.

## hashlib module

It has all necessary hash functions - rmd160, sha256, sha512. It also includes one-line functions for pbkdf2-hmac-sha512 required by bip39 and hmac-sha512 required by bip32.

## secp256k1 module

Binding to the secp256k1 c library. Global context is used. Otherwise API is exactly the same as in secp256k1.h file.
