# TISC: TEE-Isolated Stack Canary on Arm TrustZone

## GCC

This repository provides GCC with `-fstack-protector-tee` option, which enables isolation of stack canary with Trusted Execution Environment (TEE)
on Arm TrustZone.

Refer to [this repository](https://github.com/flyahn06/tz-gcc/) for further information.

### Build

```shell
mkdir gcc/build
cd gcc/build
../configure \
  --disable-bootstrap \
  --disable-multilib \
  --enable-languages=c,c++ \
  --enable-checking=yesmake clearn
make -j`nproc`
```

> Note: `--disable-bootstrap` should be removed in production build;  
> it is for faster build during development.

### Run

Use `-fstack-protector-tee` to enable tee-isolated stack canary protection.

```c
// test.c
#include <stdio.h>

int main() {
	char arr[0x8];
	
	scanf("%s", arr);
	printf("Hello, world!\n");

	return 0;
}
```

```shell
cd gcc/build/gcc
./xgcc -B"$(pwd)/" -O0 -S ../test.c -o ../test.s -fstack-protector-strong -fstack-protector-tee
```

You are expected to see `__stack_chk_guard_tee` being called in the function prologue and epilogue.

> Note: this feature is being developed currently.

```nasm
; test.s
...
main:
    ...
	call	__stack_chk_guard_tee
	movq	%rax, -8(%rbp)
	...
	movq	-8(%rbp), %rdx
	call    __stack_chk_guard_tee
	subq	%rax, %rdx
	je	.L3
	call	__stack_chk_fail
.L3:
	leave
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
```

### Development Guide

For CLion,

1. Use `bear` to make `compile_commands.json`:

```c
cd build/
bear -- make -j$(nproc)
```

2. Make symbolic link of `compile_commands.json`:

```c
# Run on build directory
ln -s compile_commands.json ../compile_commands.json
```

3. Open `compile_commands.json` to load project.