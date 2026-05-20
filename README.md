# Spangled's Simple VM Reversed
- https://crackmes.one/crackme/66767d59e7b35c09bb267078

This repository contains my analysis of the CrackMe and VM "Spangled’s Simple VM".

## Structures
- Bytecode
```
struct Bytecode 
{                                       
    _BYTE *start_addr;
    _BYTE *end_addr;                   
};
```

- VM

```
struct VM
{
  char pad_00[24];

  _DWORD regs[4];
  _DWORD pc;

  byte is_running;

  _DWORD *key;
};
```

## Bytecode Construction
Between addresses `0x140001780` and `0x1400017AB`, we can see how the bytecode is constructed, with the user-provided key included.

Later, at address `0x1400017ED`, the constructed bytecode is copied into the bytecode structure using `memmove`.

## VM Dispatcher & Handlers
At address `0x1400018D9` we see a call to the address `0x1400011F0` which is the dispatcher of this VM.
```
lea     rdx, [rbp+0E0h+bytecode]
lea     rcx, [rbp+0E0h+vm]
call    vm_run                  ; 0x1400011F0
```

By analyzing this function we can understand the VM's internal behavior and reverse its handlers.

## VM Analysis

| Opcode | Opcode Size | Name        | Format                         | Description                              |
|--------|-------------|-------------|--------------------------------|------------------------------------------|
| 0x00   | 6 bytes     | LOAD_IMM32  | `[opcode] [dstreg] [imm32]`    | `regs[dst] = imm32`                      |
| 0x02   | 3 bytes     | ADD         | `[opcode] [dstreg] [srcreg]`   | `regs[dst] += regs[src]`                 |
| 0x06   | 2 bytes     | KEYCHECK    | `[opcode] [keyreg]`            | `regs[keyreg] == vm->key` - checks the key |
| 0x07   | 1 byte      | KEYCORRECT  | `[opcode]`                     | `prints "Key is correct."`              |
| 0x08   | 1 byte      | HALT        | `[opcode]`                     | `vm->is_running = 0` - halts the VM      |

Virtual registers are stored in vm->regs[n]

## VM Bytecode
```
LOAD_IMM32 0 KEY_INPUT
KEYCHECK 0
KEYCORRECT
HALT
```

## How to Solve the CrackMe
- The `KEYCHECK` opcode handler checks `vm->key` for the correct key.

During VM initialization, at address `0x1400016E6`, we can see the following instruction:
```
mov     dword ptr [rbp+0E0h+vm.key], 0D0332h
```

By converting the hexadecimal `0x0D0332` to decimal `852786` we find the key for the CrackMe.

## Key
- `852786`
