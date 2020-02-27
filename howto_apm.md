# How to use and implement: **APM**

*Note: this tutorial is for REAL MODE ONLY!*

## What is APM?
APM is a power managment standard developed by Intel and Microsoft. APM stands for Advanced Power Management. Though superseded by **ACPI**, it is still supported on many computers.

## What can I do with it?
You can shut down, reset, and regulate power of the computer and its devices.

## What to do to set it up?

Normally, you:
- Check if the computer supports APM.
- Disconnect any APM interface that exists.
- Connect the realmode interface.
- Tell APM that the OS supports version 1.1
- Enable power management (for *all* devices)

## How to set it up

### Installation check
This code checks if it is installed.
```asm
mov ah, 53h   ; APM command
mov al, 00h   ; Installation check
xor bx, bx    ; DeviceID = 0 (APM BIOS)
int 15h       ; Call thru int 15
jc error      ; Error if carry set
```
On return the BIOS returns:
  - AH = major version (BCD)
  - AL = minor version (BCD)
  - BH = "P"
  - BL = "M"
  - CX = flags (see documentation)

### Disconnecting from interface
This code disconnects from an interface.
```asm
mov ah, 53h   ; APM command
mov al, 04h   ; Interface disconnect
xor bx, bx    ; DeviceID = 0
int 15h       ; Call thru int 15
jc disc_error ; Error
jmp good      ; No error
disc_error:
  cmp ah, 03h ; Is it because no interface was set up in the first place?
  jne error   ; No it's a real error
good:         ; We can continue
```

### Connecting to an interface
This code connects to an interface.
```asm
mov ah, 53h   ; APM command
mov al, 01h   ; Real mode interface
xor bx, bx    ; DeviceID = 0
int 15h       ; Call thru int 15
jc error      ; Error
```

### Setting APM version
This code tells the interface that the OS supports 1.1.
```asm
mov ah, 53h   ; APM command
mov al, 0eh   ; Set OS supported version
xor bx, bx    ; DeviceID = 0
mov ch, 01h   ; Major version (1)
mov cl, 01h   ; Minor version (1)
int 15h       ; Call thru int 15
jc error      ; Error if carry set
```

### Enabling power management
This code enables power management for **all** devices (it's way faster than individually)
```asm
mov ah, 53h   ; APM command
mov al, 08h   ; Change power management state
mov bx, 0001h ; On all devices
mov cx, 0001h ; Power management on (not off)
int 15h       ; Call thru int 15
jc error      ; Error if carry set
```

## What to do to use it?
Ok, now you have a fully-functional APM interface. Now, it's time to start managing power!

### Complete shutdown
;Set the power state for all devices to **OFF**
```asm
mov ah, 53h   ; APM command
mov al, 07h   ; Set the power state
mov bx, 0001h ; On all devices
mov cx, 3     ; To OFF
int 15h       ; Call thru int 15
; You should never get here!
jc error      ; Error if carry set
```

### Standby,suspend,etc
To change all devices to standby, just change `asm cx` to `asm 01h`. Likewise, to change all devices to suspend, change `asm cx` to `asm 02h`. To change a certain device's power state, look in the APM spec to find the ID, then change `asm bx` to the ID.

## What next?
You can continue on, read the spec and support lots of things, including APM 1.2, or you could support its sucessor, ACPI. But this code can support the basics of power managment, so try it out!

## I'm too lazy to write a driver myself
Sure, here's some code!

```asm

; IN: Nothing
; OUT: carry set if error
setup_apm:
  pusha
  
  mov ah, 53h   ; APM command
  mov al, 00h   ; Installation check
  xor bx, bx    ; DeviceID = 0 (APM BIOS)
  int 15h       ; Call thru int 15
  jc error      ; Error if carry set

  mov ah, 53h   ; APM command
  mov al, 04h   ; Interface disconnect
  xor bx, bx    ; DeviceID = 0
  int 15h       ; Call thru int 15
  jc disc_error ; Error
good:
  
  mov ah, 53h   ; APM command
  mov al, 01h   ; Real mode interface
  xor bx, bx    ; DeviceID = 0
  int 15h       ; Call thru int 15
  jc error      ; Error
  
  mov ah, 53h   ; APM command
  mov al, 0eh   ; Set OS supported version
  xor bx, bx    ; DeviceID = 0
  mov ch, 01h   ; Major version (1)
  mov cl, 01h   ; Minor version (1)
  int 15h       ; Call thru int 15
  jc error      ; Error if carry set
  
  mov ah, 53h   ; APM command
  mov al, 08h   ; Change power management state
  mov bx, 0001h ; On all devices
  mov cx, 0001h ; Power management on (not off)
  int 15h       ; Call thru int 15
  jc error      ; Error if carry set
  
  popa
  clc
  ret
disc_error:
  cmp ah, 03h ; Is it because no interface was set up in the first place?
  je good     ; No interface, keep going
error:       ; No, there was a real error
  popa
  stc
  ret

; IN: Nothing
; OUT: Nothing (because system is shut down)
apm_shutdown:
  pusha
  mov ah, 53h   ; APM command
  mov al, 07h   ; Set the power state
  mov bx, 0001h ; On all devices
  mov cx, 3     ; To OFF
  int 15h       ; Call thru int 15
  ; You should never get here!
error:
  popa          ; No point in setting carry,
  ret           ; if you get here there is an error
```

Well, that's all for now! Read some more tutorials!
