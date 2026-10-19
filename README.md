![libeot](docs/header.svg)

# libeot

**A small C library (and `eot2ttf` tool) that unpacks Embedded OpenType fonts into plain TrueType.**

> **Fork note:** this is a fork of [umanwizard/libeot](https://github.com/umanwizard/libeot), written by Brennan T. Vincent with contributions from David Tardon. All credit goes to the original authors; this fork only adds documentation.

## What is EOT?

Embedded OpenType (`.eot`) is Microsoft's web-font format from the Internet Explorer era. It wraps a TrueType font with a header (names, permissions, the sites allowed to use it) and can obfuscate and compress the font data. Modern browsers use WOFF/WOFF2 instead, but old sites and documents still carry `.eot` files.

## What it does

libeot reads an `.eot` file, parses its header, undoes the encoding and gives you back a normal `.ttf` font you can open or convert with any font tool.

## Features

- Parses EOT headers, versions **0x00010000, 0x00020001 and 0x00020002** (EOT v1, v2, v3)
- Exposes the metadata: family/style/full/version names, weight, italic, PANOSE, charset, Unicode and code-page ranges, embedding permissions, allowed root URLs, EUDC info
- Removes **XOR obfuscation** (`TTEMBED_XORENCRYPTDATA`, key `0x50`)
- Decompresses **MicroType Express (MTX)** fonts: LZCOMP + Compact Font Format (CTF) back to TrueType
- Writes the result to a `FILE *` or a memory buffer
- `eot2ttf` command-line converter
- `pkg-config` file (`libeot.pc`)

Not supported: AccuType (ACT) compression and writing new `.eot` files.

## Build

Needs a C99 compiler and autotools (`autoconf`, `automake`, `libtool`).

1. `autoreconf --install`
2. `./configure`
3. `make`

`make install` installs `libeot`, the headers under `include/libeot/` and the `eot2ttf` binary.

## Usage

### Command line

```sh
eot2ttf myfont.eot myfont.ttf
```

### C API

```c
#include <stdio.h>
#include <libeot/libeot.h>

/* eot: the whole .eot file in memory, eotSize: its length in bytes */
int convert(const uint8_t *eot, unsigned eotSize)
{
  struct EOTMetadata meta;
  uint8_t *ttf;
  unsigned ttfSize;

  enum EOTError err = EOT2ttf_buffer(eot, eotSize, &meta, &ttf, &ttfSize);
  if (err != EOT_SUCCESS) {
    EOTprintError(err, stderr);
    return 1;
  }

  /* ... use ttf / ttfSize, read meta.familyName, meta.weight, ... */

  EOTfreeBuffer(ttf);
  EOTfreeMetadata(&meta);
  return 0;
}
```

Link with `$(pkg-config --cflags --libs libeot)`.

| Function | What it does |
| --- | --- |
| `EOT2ttf_file(font, size, &meta, FILE *out)` | Convert and write the TTF to a file |
| `EOT2ttf_buffer(font, size, &meta, &out, &outSize)` | Convert into a new buffer (free with `EOTfreeBuffer`) |
| `EOTfillMetadata(bytes, length, &meta)` | Parse only the header |
| `EOTgetMetadataLength(bytes)` | Size of the EOT header |
| `EOTcanLegallyEdit(&meta)` | Check the embedding permissions allow editing |
| `EOTfreeMetadata(&meta)` | Free the strings inside the metadata |
| `EOTprintError(err, FILE *)` | Print a readable error message |

Headers: `libeot/libeot.h` (conversion), `libeot/EOT.h` (metadata), `libeot/EOTError.h` (error codes).

## License

[Mozilla Public License 2.0](LICENSE). The MTX decompressor in `src/lzcomp` comes from Monotype's MTX reference code and was relicensed under the MPL (see the notice at the top of `LICENSE`). Please also read [PATENTS](PATENTS) about the Microsoft/Monotype patents around EOT.

## Credits

- Original author: **Brennan T. Vincent** ([umanwizard/libeot](https://github.com/umanwizard/libeot))
- Contributor: **David Tardon**
- MTX/LZCOMP code: **Monotype Imaging**
