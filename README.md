ARM Thumb output
==============

To use this:

* Configure it with cross-compilation enabled, then use any ARM target
* I've been using `./arm-linux-gnueabi-tcc -nostdinc -nostdlib -c test.c -o test.o` to compile, and `arm-none-eabi-objdump -S test.o` to dump the output
* I was comparing it with what I got from `arm-none-eabi-gcc -mlittle-endian -mthumb -mcpu=cortex-m3  -mfix-cortex-m3-ldrd -mfloat-abi=soft -nostdinc -nostdlib -c test.c -o test.o`

In terms of implementation:

* Everything is in `arm-gen.c`.
* `tcc-gen.c` has a single mod, which adds one to the symbol address (signifying that the symbol is Thumb, not ARM). There must be a nicer way of doing this, but hey - proof of concept.
* I added `ot()` which outputs a thumb instruction - the original `o()` would output a 32 bit ARM instruction - so every call to `o()` needs replacing with a thumb equivalent
* The function prolog/epilog is hacked up at the moment so only works with very basic stuff (no stack pushing)
* `encbranch`/`decbranch`/`gsym_addr` **do** seem to work, but encbranch won't work reliably for conditional branches as these have less bits for the address

Would be nice:

* Ideally we'd start out with a new `thumb-gen.c` containing stubs - it would get rid of the [RELICENSING](RELICENSING) issue
* Find a way to separate ARM and Thumb code gen (ideally separate files), and maybe make TCC respect the `-mthumb` switch like GCC does (rather than having a separate binary).


### test.sh

```
#!/bin/bash
arm-none-eabi-gcc -mlittle-endian -mthumb -mcpu=cortex-m3  -mfix-cortex-m3-ldrd -mfloat-abi=soft -nostdinc -nostdlib -c test.c -o test.o 
echo ------------------------------------------------
echo   GCC
echo ------------------------------------------------
arm-none-eabi-objdump -S test.o
mv test.o test-gcc.o


./arm-linux-gnueabi-tcc -nostdinc -nostdlib -c test.c -o test.o 
echo ------------------------------------------------
echo   TCC
echo ------------------------------------------------
arm-none-eabi-objdump -S test.o
mv test.o test-tcc.o
```

### test.c

```
// test.c
int foobar() {
  return 4;
}
```


### Original README

is [here](README)
