# Swap Fn and LeftControl on ThinkPad Compact USB Keyboard with TrackPoint

## Steps

- Download the firmware update tool for FW ver. 3.30 from Lenovo's homepage (sha256sum of `tp_compact_usb_kb_with_trackpoint_fw.exe` is 7116a3819ee094857d21e4671cb6cf953d582372126f0f6728f6b2421eda7bd4)
- Open the file in the hex editor of your choice
- Change position `0x74004` (`475140` decimal) from `0xba` to `0xf5` (this changes the function of the "LeftControl" key to "Fn"
- Change position `0x740BA` (`475322` decimal) from `0xf5` to `0xba` (this changes the function of the "Fn" key to "LeftControl"
- The sha256sum of the modified file is `123143092dab578550c87a62526b07a6c5f06c047f2455be87971aa51577e300`
- Use this file to update your keyboard's firmware

Note: It worked for me, but I will not guarantee that it won't brick your keyboard. Use at your own risk!

### More detailed description

I used the info in this issue and https://github.com/vpelletier/dissn8 to get a human readable assembler file of the firmware

After a lot of to and for I found out that there is a table in the ROM at word `0x05a0` that translates the keyboard matrix to Usage IDs as specified in "Universal Serial Bus HID Usage Tables", rev. 1.12 (October 28, 2004), table 12 from the USB Implementer's forum.

The actual translation is done by the function at word 0x0186 (label `func_0186` when using the aforementioned disassembler. Handling of the "Fn" key is at label `func_0186_015f` and following.

But for swapping "Fn" and "LeftControl" we need to modify the translation table at word `0x05a0`:
The "Fn" key is encoded as `0x00af` at word `0x0615`, i.e. byte `0xc2a` (`0x740ba` in the .exe), since it is XORed with `0x5a` the value in the .exe is `0xf5`. To make the "Fn" key work as "LeftControl", it must be changed to `0xe0 ^ 0x5a = 0xba`.
The "LeftControl" key is encoded as `0x00e0` at word `0x05ba`, i.e. `0xb74` bytes (`0x74004` in the .exe), since it is XORed with `0x5a` the value in the .exe is `0xba`. To make the "LeftControl" key work as "Fn", it must be changed to `0xaf ^ 0x5a = 0xf5`.

## How to run the firmware on Linux in a Virtualbox

1. Run linux with `ibt=off` kernel parameter. Note: This might not be necessary with newer kernels.
2. Start Windows 11 Virtualbox with USB 2.0 controller.
3. Install modified firmware to keyboard.

## Sources

- https://github.com/lentinj/tp-compact-keyboard/issues/32
- https://www.reddit.com/r/thinkpad/comments/i43c3h/comment/gefu016/?utm_source=share&utm_medium=web2x&context=3
