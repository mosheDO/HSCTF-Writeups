# Jumper (Binary Exploitation, 150 points)

### If you haven't read the writeup on [Review](../review-100/README.md), you should do so to get a quick primer on buffer overflows.

Armed with our new knowledge about buffer overflows, let's take a look at jumper. 

```
#include <stdlib.h>
#include <string.h>
#include <stdio.h>

void lol() {
	//get flag
	system("/bin/cat /home/ctfadmin/HSCTF-Problems/jumper/flag" /*|        | o */);
}						  										/*|        |/|\ look it's keith! */
						  										/*|        |/ \ great jumper, he is*/
						  										/*|        | */
void jumper() {	

	// can overflow!!	  										/*|        | */
	char dest[32];		  										/*|        | */
	printf("Gimme some input: ");								/*|        | */

	//dangerous!
	gets(dest);			  										/*|        | */
	printf("Jumping to %s\n", dest);							/*|        | */
}						  										/*|        | */
						  										/*|        | */
int main() {													/*|        | */
	setbuf(stdout, NULL);			  							/*|        | */
	gid_t gid = getegid();										/*|        | */
	setresgid(gid,gid,gid);										/*|        | */
	jumper();			  										/*|        | */
}						  										/*|        | */
```
Indeed, we spy another gets() call. However, this time there doesn't seem to be anything to overwrite after `dest`, so what do we do from here?

Let's go back to the Wikipedia diagrams. What's located below our character arrays?

![over1.png](over1.png)

Cool! Maybe if we overwrite the return address of jumper(), we can redirect program flow as long as there aren't any checks done. So now, our theoretical payload looks like `'AAAAA....' (garbage input) + '\x05\x05\x05\x05' (address we want to jump to)`. 

#### A note must be made here: most architectures store addresses in *little endian*: that is, the address `0xA1B2C3D4` is stored as `\xD4\xC3\xB2\xA1` in memory. More research is left as an exercise to the reader.

However, how are we going to find the address of lol() so that we can jump to it? Here begins the use of Linux utilities **objdump** and **gdb**. At its core, `objdump` provides a raw disassembly of an ELF binary, and `gdb` allows you to debug a program by stepping through instructions. 

Let's begin by running `objdump -d jumper -M intel`. This will (-d)isassemble jumper using Intel syntax.

![obj.png](obj.png)

By scrolling up a bit, we find the addresses of both lol() and jumper(). We can see that lol() starts at address `0x0804852b`, so let's modify our payload. It no wlooks like `'AAAA....' + '\x2b\x85\x04\x08' (little endian!)`.

