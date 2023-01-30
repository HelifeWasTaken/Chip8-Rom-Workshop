# Chip8-Rom-Workshop
Learn Chip-8 Rom dev

# Chip8 rom programming tutorial

Here we start as low as possible, in my next workshops I will show you how to graphics computing either with n64 SDK or with OpenGL.

## Preparation

First of all, you need to clone this repository it will feature an emulator written in C with SDL and a hex editor written in python,
to write your programs.

```bash
./install.sh # Will install the dependencies for fedora and compile the emulator

./chip8 # Will run the emulator you will be then asked to enter the path to the rom you want to run

python3 hex_editor.py file_name # Will run the hex editor
```

## The chip8

The chip8 is a virtual machine that was created in the 70's, it was used to run games on calculators and other devices that didn't have a lot of memory.

It has 0x23 opcodes, 4K of memory, 0x10 registers, a stack, a timer and a sound timer.

The chip8 has a 64x32 screen, it is black and white, the screen is divided in 8x8 pixels blocks, each block is called a sprite.

It has 16 keys, they are mapped like this:
```
1 2 3 C
4 5 6 D
7 8 9 E
A 0 B F
```

## The memory

The chip8 has 0x10 registers, they are named V0 to VF, the first 0xF registers are general purpose, the last one is used as a flag, the registers are used to store data such as variables.

The chip8 has 4K of memory, the first 0x200 bytes are reserved for the interpreter, the rest is used to store the program and the sprites.
It means that your program starts at 0x200 in hexadecimal.

That also means that the chip8 can only run programs that are smaller than 0xE00 bytes.

The chip8 has a stack, it can store 0x10 addresses, it is used to store the address of the next instruction when a subroutine is called.

The chip8 has 0x2 timers, the delay timer and the sound timer, they are used to make sounds and to make animations.

## The opcodes

The opcodes are 16 bits long, they are divided in 4 parts, the first part is the instruction, the second part is the first argument, the third part is the second argument and the last part is the last argument.

Here is a table of the opcodes: (Of course you won't need to know all of them, but it's good to know what they do)

| Instruction | Name | Description | C equivalent |
|-------------|------|-------------|--------------|
| 0NNN | SYS | Call RCA 1802 program at address NNN | |
| 00E0 | CLS | Clear the display | clear_screen(BLACK) |
| 00EE | RET | Return from a subroutine | return |
| 1NNN | JP | Jump to address NNN | goto NNN |
| 2NNN | CALL | Call subroutine at NNN | *(0xNNN)() |
| 3XNN | SE | Skip the next instruction if VX equals NN | if(VX == NN) goto PC + 2 |
| 4XNN | SNE | Skip the next instruction if VX doesn't equal NN | if(VX != NN) goto PC + 2 |
| 5XY0 | SE | Skip the next instruction if VX equals VY | if(VX == VY) goto PC + 2 |
| 6XNN | LD | Set VX to NN | VX = NN |
| 7XNN | ADD | Add NN to VX | VX += NN |
| 8XY0 | LD | Set VX to the value of VY | VX = VY |
| 8XY1 | OR | Set VX to VX or VY | VX |= VY |
| 8XY2 | AND | Set VX to VX and VY | VX &= VY |
| 8XY3 | XOR | Set VX to VX xor VY | VX ^= VY |
| 8XY4 | ADD | Add VY to VX, set VF to 1 if there is a carry, set VF to 0 if there isn't | VX += VY; VF = (VX > 0xFF) |
| 8XY5 | SUB | Subtract VY from VX, set VF to 0 if there is a borrow, set VF to 1 if there isn't | VX -= VY; VF = (VX < 0) |
| 8XY6 | SHR | Shift VX right by one, set VF to the least significant bit of VX before the shift | VF = VX & 0x1; VX >>= 1 |
| 8XY7 | SUBN | Set VX to VY minus VX, set VF to 0 if there is a borrow, set VF to 1 if there isn't | VX = VY - VX; VF = (VX < 0) |
| 8XYE | SHL | Shift VX left by one, set VF to the most significant bit of VX before the shift | VF = VX & 0x80; VX <<= 1 |
| 9XY0 | SNE | Skip the next instruction if VX doesn't equal VY | if(VX != VY) goto PC + 2 |
| ANNN | LD | Set I to the address NNN | I = NNN |
| BNNN | JP | Jump to the address NNN plus V0 | goto NNN + V0 |
| CXNN | RND | Set VX to a random number and NN | VX = rand() & NN |
| DXYN | DRW | Draw a sprite at coordinate (VX, VY) that has a width of 8 pixels and a height of N pixels. Each row of 8 pixels is read as bit-coded starting from memory location I; I value doesn't change after the execution of this instruction. VF is set to 1 if any screen pixels are flipped from set to unset when the sprite is drawn, and to 0 if that doesn't happen | draw_sprite(VX, VY, N) |
| EX9E | SKP | Skip the next instruction if the key stored in VX is pressed | if(key_pressed(VX)) goto PC + 2 |
| EXA1 | SKNP | Skip the next instruction if the key stored in VX isn't pressed | if(!key_pressed(VX)) goto PC + 2 |
| FX07 | LD | Set VX to the value of the delay timer | VX = get_delay() |
| FX0A | LD | Wait for a key press, store the value of the key in VX | VX = wait_key() |
| FX15 | LD | Set the delay timer to VX | set_delay(VX) |
| FX18 | LD | Set the sound timer to VX | set_sound(VX) |
| FX1E | ADD | Add VX to I but VF is not affected | I += VX |
| FX29 | LD | Set I to the location of the sprite for the character in VX | I = get_sprite(VX) |
| FX33 | LD | Store the binary-coded decimal representation of VX at the addresses I, I + 1 and I + 2 | *(I) = VX / 100; *(I + 1) = (VX / 10) % 10; *(I + 2) = VX % 10 |
| FX55 | LD | Store V0 to VX in memory starting at address I | for(int i = 0; i <= X; i++) *(I + i) = V[i] |
| FX65 | LD | Fill V0 to VX with values from memory starting at address I | for(int i = 0; i <= X; i++) V[i] = *(I + i) |

About **8XY4**: Technically VX cannot be greater than 0xFF because of the 8 bits register but the carry flag is set to 1 if there is a carry, so if VX is 0xFF and VY is 0x01, VX will be 0x00 and the carry flag will be set to 1.

About **8XY5**: Technically VX cannot be less than 0 because of the 8 bits register but the borrow flag is set to 0 if there is a borrow, so if VX is 0x00 and VY is 0x01, VX will be 0xFF and the borrow flag will be set to 0.

About **8XY7**: Technically VX cannot be less than 0 because of the 8 bits register but the borrow flag is set to 0 if there is a borrow, so if VX is 0x01 and VY is 0x00, VX will be 0xFF and the borrow flag will be set to 0.

## Writing a program

The first thing you need to do is to write your program, you can use the hex editor to do so, you can also use an external hex editor
like [HxD](https://mh-nexus.de/en/hxd/) or [010 Editor](https://www.sweetscape.com/010editor/).

The hex editor will display the memory of the chip8, you can write your program in hexadecimal,
the first 512 bytes are not present in the memory and will be created by the emulator when you run your program.

In order to be able to demonstrate pseudo code I will use [Intel syntax](https://en.wikipedia.org/wiki/X86_assembly_language#Syntax) for the assembly code.

### 1. Write a program that loops forever

This is the simplest program you can write, it will just loop forever and do nothing.
For now I will give you the pseudo code:
```asm
:main
    jmp main
```

Do not forget that your **main** starts at `0x200` and not at `0x000`.

### 2. Write a program that draws a sprite at (0, 0) then loops forever

This program will draw a sprite at (0, 0) and then loop forever.

First we need to know how to draw a sprite, the instruction to draw a sprite is `DXYN` where `X` and `Y` are the coordinates of the sprite and `N` is the height of the sprite.

But before we can draw a sprite we need to know how to store a sprite in memory.

A sprite is a 8xN pixel image, each pixel can be either on or off, so we can store a sprite in memory by using 1 byte for each row of the sprite.

A full sprite would be of value `0xFF` and an empty sprite would be of value `0x00`.

So in order to draw a sprite we need to store it in memory then set the `I` register to the address of the sprite and then draw it.

Here is the pseudo code:
```asm
:main
    v1 := 0x00
    v2 := 0x00
    i := :pixel
    draw v1, v2, 1

:infinite_loop
    jmp infinite_loop

:pixel 0xFF
```

Do not forget that every time you add or remove an instruction your offset to the sprite will change.

### 3. Write a program (explanation below):
In a loop:
    draw a sprite at (0, 0).
    wait for a key press.
    hide the sprite.
    wait for a key press.
    loop back to the beginning.

```asm
:main
    v1 := 0x00
    v2 := 0x00
    i := :pixel
    draw v1, v2, 1
    wait_key
    draw v1, v2, 0
    wait_key
    jmp main

:pixel 0xFF
```

### 3. Write a program that draws a sprite at (0, 0) then let's you move with the arrow keys

This program will draw a sprite at (0, 0) and then let's you move with the arrow keys.

First we need to know how to move a sprite, we can move a sprite by changing the coordinates of the sprite.

So in order to move a sprite we need to store the coordinates of the sprite in memory then set the `VX` and `VY` registers to the coordinates of the sprite and then draw it.

Here is the pseudo code:
```asm
; assuming wasd for the arrow keys
; w -> 0x5
; a -> 0x7
; s -> 0x8
; d -> 0x9

:main
    v2 := 0x00 ; x coordinate
    v3 := 0x00 ; y coordinate
    i := :pixel ; set the i register to the address of the sprite
    draw v2, v3, 1 ; draw the sprite at (0, 0)

:infinite_loop
    v1 := wait_key() ; wait for a key press
    draw v2, v3, 1 ; draw the sprite to the old position to erase it
                   ; this works by xor-ing the sprite with itself
                   ; so you do not need to clear the screen
                   ; you can also use the clear screen instruction
                   ; but it is slower

    se v1, 0x7 ; if the key pressed is the left arrow
    jmp :move_right ; move the sprite to the right
    sub v2, 0x01 ; decrease the x coordinate by 1
    jmp draw_at_new_position ; draw the sprite at the new position

:move_right

    se v1, 0x9 ; if the key pressed is the right arrow
    jmp :move_up; move the sprite to the up
    add v2, 0x01 ; increase the x coordinate by 1
    jmp draw_at_new_position ; draw the sprite at the new position

:move_up
    se v1, 0x5 ; if the key pressed is the up arrow
    jmp draw_at_new_position ; draw the sprite at the new position
    sub v3, 0x01 ; decrease the y coordinate by 1
    jmp draw_at_new_position ; draw the sprite at the new position

:move_down
    se v1, 0x8 ; if the key pressed is the down arrow
    jmp draw_at_new_position ; draw the sprite at the new position
    add v3, 0x01 ; increase the y coordinate by 1

:draw_at_new_position
    draw v2, v3, 1 ; draw the sprite at the new position
    jmp :infinite_loop ; loop forever

:pixel 0xFF
```

```octo
# johnearnest.github.com/Octo
# Chip-8 emulator

: main
	v2 := 0
	v3 := 0
	i := pixel
	sprite v2 v3 1
	
	loop
		v1 := key
		sprite v2 v3 1
		
		if v1 != 0x7 then jump move_right
		v2 -= 1
		jump redraw
: move_right
		if v1 != 0x9 then jump move_up
		v2 += 1
		jump redraw
: move_up
		if v1 != 0x5 then jump move_down
		v3 -= 1
		jump redraw
: move_down
		if v1 != 0x8 then jump redraw
		v3 += 1
: redraw
		sprite v2 v3 1
again

: pixel
  0xFF
```
