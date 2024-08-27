# oledd

The `oledd` is a daemon running as a service to provide the little menu and
draws to the display. It has a resource file and a corresponding config file:

- `/etc/oled_res`
- `/etc/oled_animation.config`

The init script is at `/etc/init.d/start_oledd`.

## header

The first short might be simply the sprite count (148), followed by a sprite,
14x8 black pixels.

```
00000000: 2402 0000 0000 0000 0000 1400 0800 0000  $...............
00000010: 0000 0000 0000 0000 0000 0000 0000 ffff  ................
00000020: ffff ffff ffff ffff ffff ffff ffff ffff  ................
```

## sprites

Each sprite starts with 14 shorts of metadata, and sprites are different sizes.
The image encoding is just one bit per pixel, and no compression.

From the following examples, we recognize `0x0080` (`128`), i.e. the screen
width/height. Reversing the `oledd` binary verifies the assumption.
The first byte identifies the sprite.

The sprite ID is used in `image_list` in the config file, which is mostly
understandable.

For example:
```
fa_start
fa_id=30010;
x=0;
y=0;
z=0;
block_id=0;
duration=1000;
frame_interval=1;
repeat=true;
fa_name=bootup;
image_list=100,101,102,103,104,105,106,107,108,109,110,111;
fa_end
```

Here are two samples that are part of the startup animation:

```
00000030: ffff 6400 0000 0000 0000 8000 8000 0100  ..d.............
00000040: 6400 6400 0000 0000 0000 0000 0000 ffff  d.d.............
00000050: ffff ffff ffff ffff ffff ffff ffff ffff  ................
```

```
6400 0000 0000 0000 8000 8000 0100 6400 6400 0000 0000 0000 0000 0000
```

```
00000840: ffff ffff ffff ffff ffff ffff ffff 6500  ..............e.
00000850: 0000 0000 0000 8000 8000 0100 6400 6400  ............d.d.
00000860: 0000 0000 0000 0000 0000 ffff ffff ffff  ................
```

```
6500 0000 0000 0000 8000 8000 0100 6400 6400 0000 0000 0000 0000 0000
```

```rs
struct SpriteMeta {
    sprite_id: u16,
    _1: u16,
    _2: u16,
    _3: u16,
    width: u16,
    height: u16,
    _4: u16,
    _5: u16,
    _6: u16,
    _7: u16,
    _8: u16,
    _9: u16,
    _10: u16,
    _11: u16,
}
```

## conversion

note: `128*128 bit = 128*128/8 bytes = 2048 bytes`

- to bmp

```sh
dd if=oled_res bs=1 skip=78 count=2048 of=load_anim_100.res
convert -size 128x128 -negate -depth 1 gray:load_anim_100.res load_anim_100.bmp
```

Now you can edit the bitmap file.

- back to raw 

```sh
convert load_anim_100.bmp -negate -depth 1 gray:load_anim_100.raw
```

Let's say we did the above for the first two sprites.

- patch the resource file

```sh
cp oled_res oled_res.hexed # keep a copy of the original
dd if=load_anim_100.raw bs=1 seek=78 conv=notrunc of=oled_res.hexed
dd if=load_anim_101.raw bs=1 seek=2154 conv=notrunc of=oled_res.hexed
```

And `adb push oled_res.hexed /etc/oled_res`. Enjoy! :tada: