# How to use and implement: Rebooting
Rebooting is basically resetting. There are lots of ways to do this.

1. Use the ACPI's reset command



Yes, this works very well. Unfortunately, you have to implement ACPI, which is very (in my point of view) complicated.(1)

2. Use the ResetSystem UEFI service



This also works, if you have an OS that supports UEFI. But in this tutorial, we will not cover this.

3. Load a zero-sized IDT, then issue an interrupt (it will triple fault and reset)



This *does* work, but it is very "hacky". There are more reliable ways to do this.

4. Use the 8042 keyboard to pulse the CPU's RESET pin.



Here is some code:
```asm
reset:
  mov al, 0x02
reset_loop:
  in al, 0x64
  test al, 0x02
  jnz reset_loop
continue:
  out 0x64, 0xFE
  hlt
```

## What's next?
Probably add this to your power management driver.


Well, that's all for now! Read some more tutorials! =)
