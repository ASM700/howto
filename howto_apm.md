# How to use and implement: **APM**

*Note: this tutorial is for REAL MODE ONLY!*

## What is APM?
APM is a power managment standard developed by Intel and Microsoft. APM stands for Advanced Power Management.

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
