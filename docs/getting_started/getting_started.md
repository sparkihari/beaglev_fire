# Getting Started with Hello World using C on BeagleV Fire Board

A quick guide to writing, compiling, and running a simple C program on BeagleV Fire Board.


## 1. Log in to your BeagleV Fire via SSH
 
```bash
ssh username@server_ip_or_hostname
```

## 2. Verify the C compiler (if not already installed)

Verify it installed correctly:

```bash
gcc --version
```

## 3. Create the source file

Create a file named `helloworld.c`:

```bash
nano helloworld.c
```

(You can also use `vim`, or any text editor you prefer.)

Add the following code:

```c
#include <stdio.h>

int main(void) {
    printf("Hello World!\n");
    return 0;
}
```

Save and exit:
- In `nano`: press `Ctrl+O`, then `Enter`, then `Ctrl+X`.

## 4. Compile the program

Use `gcc` to compile `helloworld.c` into an executable:

```bash
gcc helloworld.c -o helloworld
```

This creates an executable file named `helloworld` in the current directory.

## 5. Run the program

```bash
./helloworld
```

Expected output:

```
Hello World!
```

