# nanopy
* Install by running `pip install nanopy`.
* Point to a custom compiler (default is `gcc`) by prepending the installation command with `CC=path/to/custom/c/compiler`.
  * When using `Visual C`, additionally prepend the installation command with `USE_VC=1`.
* For GPU, appropriate OpenCL ICD and headers are required. `sudo apt install ocl-icd-opencl-dev amd/intel/nvidia-opencl-icd`
  * Enable GPU usage by prepending the installation command with `USE_GPU=1`.

[![Build Status](https://travis-ci.com/npy0/nanopy.svg?branch=master)](https://travis-ci.com/npy0/nanopy) [![docs](https://readthedocs.org/projects/nanopy/badge/)](http://nanopy.rtfd.io) [![PyPI](https://img.shields.io/pypi/v/nanopy)](https://pypi.org/project/nanopy) [![PyPI - Implementation](https://img.shields.io/pypi/implementation/nanopy)](https://pypi.org/project/nanopy) [![PyPI - Downloads](https://img.shields.io/pypi/dm/nanopy)](https://pypistats.org/packages/nanopy) [![PyPI - License](https://img.shields.io/pypi/l/nanopy)](https://opensource.org/licenses/MIT)

## Usage
* Functions in the core library are written in the same template as [nano's RPC protocol](https://docs.nano.org/commands/rpc-protocol/). If there is a function, you can more or less call it the same way you would get that `action` done via RPC. For e.g., the RPC `action` to generate work for a hash is, [work_generate](https://docs.nano.org/commands/rpc-protocol/#work_generate) with `hash` as a parameter. In the library, the `action` becomes the function name and parameters become function arguments. Thus to generate work, call `work_generate(hash)`.
  * Optional RPC parameters become optional function arguments in python. In `work_generate`, `use_peers` and `difficulty` are optional arguments available for RPC. However, `use_peers` is not a useful argument for local operations. Thus only `difficulty` is available as an argument. It can be supplied as `work_generate(hash, difficulty=x)`.
  * Only purely local `action`s are supported in the core library (work generation, signing, account key derivations, etc.).
* Functions in the `rpc` sub-module follow the exact template as [nano's RPC protocol](https://docs.nano.org/commands/rpc-protocol/). Unlike the core library, there is no reason to omit an `action` or parameter. Thus the library is a fully compatible API to nano-node's RPC.
* [nano's RPC docs](https://docs.nano.org/commands/rpc-protocol/) can be used as a manual for this library. There are no changes in `action` or `parameter` names, except in a few cases \(`hash`, `id`, `type`, `async`\) where the parameter names are keywords in python. For those exceptions, arguments are prepended with an underscore \(`_hash`, `_id`, `_type`, `_async`\).

## Wallet
Although not part of the package, the light wallet included in the repository can be a reference to understand how the library works.

### Wallet options
* The wallet looks for default configuration in `$HOME/.config/nanopy/<network>.conf`.
  * `<network>` is one of nano, banano, or beta. nano is the default.
  * Default mode of operation is to check state of all accounts in `$HOME/.config/nanopy/<network>.conf`.
* `--new`. Generate a new seed and derive index 0 account from it.
  * Seeds are generated using `os.urandom()`
  * Generated seeds are stored in a GnuPG AES256 encrypted file.
  * AES256 encryption key is 8 bytes salt + password stretched with 65011712 rounds of SHA512. Adjust options by editing `$HOME/.config/nanopy/gpg.conf`.
  * Wallets can also be made by encrypting a file that has a seed using the command, `gpg -ca FILE`.
  * Options used by the encryption can be verified by inspecting the header in the gpg file. `gpg -v --list-packets FILE`. `cipher 9` is AES256. `s2k 3` is iterated and salted key derivation mode. `hash 10` corresponds to SHA512. `count` is the number of iterations (max 65011712).
  * To get the seed, `gpg -d FILE`
* `-a`, `--audit-file`. Check state of all accounts in a file.
* `-b`, `--broadcast`. Broadcast a block in JSON format. Blocks generated on an air-gapped system using `--offline` tag can be broadcast using this option.
* `-n`, `--network`. Choose the network to interact with - nano, banano, or beta. The default network is nano.
* `-t`, `--tor`. Communicate with RPC node via the tor network.
* `-d`, `--demo`. Run in demo mode.

The wallet has a sub-command, `nanopy-wallet open FILE`, to unlock previously encrypted seeds. `open` has the following options.
* `-i`, `--index`. Index of the account unlocked from the seed. (Default=0)
* `-s`, `--send`. Supply destination address to create a send block.
* `-e`, `--empty`. Empty out funds to the specified send address.
* `-r`, `--rep`. Supply representative address to change representative.
  * Change representative tag can be combined with send and receive blocks.
* `-a`, `--audit`. Check state of all accounts from index 0 to the specified limit. (limit is supplied using the `-i` tag)
* `-p`, `--password`. Change password of the encrypted wallet.')
* `--offline`. Generate blocks in offline mode. In the offline mode, current state of the account is acquired from the default configuration in `$HOME/.config/nanopy/<network>.conf`. Refer to the sample file for more details.

## Support
Contact me on nano's [forum](https://forum.nano.org/u/npy) (`npy`) or discord (`npy#2928`). You can support the project by reporting any bugs you find and/or submitting fixes/improvements. When submitting pull requests please format the code using `black` (for Python) or `clang-format` (for C).
```
clang-format -i nanopy/*.c
black nanopy docs nanopy-wallet setup.py travis.py
```
