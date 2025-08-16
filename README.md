# emptyX.py — Create empty staidx0.mul and statics0.mul for custom UO maps

This utility generates a valid empty `staidx0.mul` and a matching empty `statics0.mul` for Ultima Online style maps. It is handy for prototyping small custom maps, bootstrapping tools, and learning how UO static layers are indexed and stored.

## How it fits together

- `map0.mul` holds terrain tiles only.  
- `art.mul` and `artidx.mul` hold the graphics for tiles and items.  
- `statics0.mul` is not graphics. It is a list of placement records that say “place TileID T at local (x, y) in this 8×8 block at elevation z with hue h.”  
- `staidx0.mul` is a block index. One 12-byte entry per 8×8 tile block gives the byte offset and length of that block’s placement records inside `statics0.mul`.  

In other words: `art.mul` is the catalog of pictures, `statics0.mul` is where those pictures are placed, and `staidx0.mul` tells you where each block’s placements live inside `statics0.mul`.

## What the script does

1. Prompts for Blocks X and Blocks Y where each block is 8×8 tiles.  
2. Computes entry count and writes `staidx0.mul` with one 12-byte entry per block using little endian layout `<iii`.  
3. Writes each index entry as `lookup = -1, length = 0, extra = 0` which means “this block has no statics.”  
4. Creates an empty `statics0.mul` file.  

Example: 16 × 16 blocks equals 128 × 128 tiles and produces 256 index entries, 3072 bytes total.

## Learning outcomes

- **Binary file structures** — learn how fixed-size records, indexes, and offsets make large worlds streamable and editable.  
- **C-style struct layouts** — see how 32-bit integers are laid out in memory and why endianness matters.  
- **Game data layering** — understand how engines layer terrain, placements, and art to render massive maps efficiently.  
- **Python systems programming** — practice byte-level file I/O and use `struct.pack`/`struct.unpack` to serialize data in a portable way.  

## Project ideas

- **Block occupancy visualizer** — parse `staidx0.mul` and show a grid heatmap of which 8×8 blocks contain static data.  
- **Minimal static injector** — extend the tool to write a few sample placement records into `statics0.mul` and update `staidx0.mul`.  
- **CSV or JSON to statics converter** — define statics in a human-friendly table, then compile to a valid `.mul` pair.  
- **Integrity checker** — scan a `.mul` pair to detect overlapping regions, invalid lengths not divisible by 7, or out-of-range offsets.  
- **Teaching demos** — use tiny maps like 8 × 8 or 16 × 16 blocks to demonstrate binary formats, indexing, and retro asset pipelines.  

## Usage

```bash
python emptyX.py
````

Follow the prompts for **Blocks X** and **Blocks Y**. The script writes `staidx0.mul` and `statics0.mul` to the current directory.

## Example run

```
== Create empty staidx0.mul/statics0.mul ==
Each block is 8×8 tiles. You will enter block counts, not tiles.

Blocks X (e.g., 16): 16
Blocks Y (e.g., 16): 16

Done.
Blocks: 16 × 16 (tiles: 128 × 128)
Entries written: 256
staidx0.mul size: 3072 bytes
Wrote: /path/to/staidx0.mul
Wrote: /path/to/statics0.mul
```

## File format reference

### staidx0.mul entry layout per block

* `int32 lookup` — little endian byte offset into `statics0.mul` or -1 for empty
* `int32 length` — length in bytes for this block’s placement list (multiple of 7)
* `int32 extra` — reserved or checksum field, commonly 0 in tooling

### statics0.mul placement record layout (7 bytes each)

* `uint16 tile_id` — which art resource to place
* `uint8 x_local` — 0 to 7 within the block
* `uint8 y_local` — 0 to 7 within the block
* `int8 z` — elevation
* `uint16 hue` — color index, 0 for default

## Requirements

* Python 3.7 or newer
* Standard library only

## License

MIT. See LICENSE.

