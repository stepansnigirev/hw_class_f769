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

# Roadmap:

- [x] `hashlib` module with `sha1`, `ripemd160`, `sha256` and `sha512` functions (C module)
- [x] `hmac` module (python)
- [x] `pbkdf2` module (python)
- [x] `_ecc` elliptic curve arithmetic (C module)
- [x] `ecc` elliptic curve library (with `__mul__` and `__add__`)
- [x] minimal `display` library
- [ ] littlevgl GUI library
- [ ] dynamic SD card (mount / unmount)
- [ ] hardware crypto accelerators support
- [ ] optimize `hmac`, `pbkdf2` with C modules

# API

## `display` library

Works on ly on STM32F769DISCO right now, and only a few functions are currently supported:
- `init()` - inits the screen and clears it. Should be run first.
- `print(msg, x, y)` - prints the `msg` string at coordinates `(x,y)`. Doesn't support `\n`.
- `clear()` - clears the screen.

## `bitcoin` library

Pure micropython bitcoin library adapted from Jimmy's [pybtcfork](https://github.com/jimmysong/pybtcfork) library. For elliptic curves it uses `_ecc` module (see below).

Currently ported `PrivateKey` and `PublicKey` classes. These classes implement `__add__`, `__mul__` and `__truediv__` methods, so normal elliptic curve arithmetics works with them. This means you can do scalar addition, multiplication and division as well as point addition and multiplication by a scalar.

Simple example:

```py
from bitcoin import *
pk = PrivateKey.parse("L4je5ce4nCaAfXD96PQzgcf7dFYDDghwQqULwLWyxjdo9KBxLFGK")
pub = pk.public_key
print(pub.address())
```

## `_ecc` C module

Minimal C module that provides a point ariphmetics functions. Can be used for efficient point addition and multiplication by a scalar.

Available functions:

- `get_public_key33(privkey)` - takes a 32-byte array with private key in big endian and returns 33-byte array with compressed sec of the public key.
- `get_public_key65(privkey)` - takes a 32-byte array with private key in big endian and returns 65-byte array with uncompressed sec of the public key.
- `point_add(p1, p2)` - takes byte arrays with sec of two points and returns a 65-byte array with sec of the sum.
- `point_multiply(scalar, point)` - takes a 32-byte array with a scalar and byte array with sec of the point and returns 65-byte array with sec of the product
- `validate_pubkey(point)` - takes a byte array with sec of the point and checks if it is on the curve or not.
