Download Link: https://assignmentchef.com/product/solved-csci2400-the-attack-lab-understanding-buffer-overflow-bugs
<br>
<h1>1          Introduction</h1>

This assignment involves generating a total of five attacks on two programs having different security vulnerabilities. Outcomes you will gain from this lab include:

<ul>

 <li>You will learn different ways that attackers can exploit security vulnerabilities when programs do not safeguard themselves well enough against buffer overflows.</li>

 <li>Through this, you will get a better understanding of how to write programs that are more secure, as well as some of the features provided by compilers and operating systems to make programs less vulnerable.</li>

 <li>You will gain a deeper understanding of the stack and parameter-passing mechanisms of x86-64 machine code.</li>

 <li>You will gain a deeper understanding of how x86-64 instructions are encoded.</li>

 <li>You will gain more experience with debugging tools such as GDB and OBJDUMP.</li>

</ul>

Note: In this lab, you will gain firsthand experience with methods used to exploit security weaknesses in operating systems and network servers. Our purpose is to help you learn about the runtime operation of programs and to understand the nature of these security weaknesses so that you can avoid them when you write system code. We do not condone the use of any other form of attack to gain unauthorized access to any system resources.

You will want to study Sections 3.10.3 and 3.10.4 of the CS:APP3e book as reference material for this lab.

<h1>2          Logistics</h1>

As usual, this is an individual project. You will generate attacks for target programs that are custom generated for you.

<h2>2.1        Getting Files</h2>

You can obtain your files by pointing your Web browser at:

http://cs2400-applied.cs.colorado.edu:15513/

The server will build your files and return them to your browser in a tar file called target<em>k</em>.tar, where <em>k </em>is the unique number of your target programs.

Note: It takes a few seconds to build and download your target, so please be patient.

Save the target<em>k</em>.tar file in a (protected) Linux directory in which you plan to do your work. Then give the command: tar -xvf target<em>k</em>.tar. This will extract a directory target<em>k </em>containing the files described below.

You should only download one set of files. If for some reason you download multiple targets, choose one target to work on and delete the rest.

Warning: If you expand your target<em>k</em>.tar on a PC, by using a utility such as Winzip, or letting your browser do the extraction, you’ll risk resetting permission bits on the executable files.

The files in target<em>k </em>include:

README.txt: A file describing the contents of the directory ctarget: An executable program vulnerable to <em>code-injection </em>attacks rtarget: An executable program vulnerable to <em>return-oriented-programming </em>attacks cookie.txt: An 8-digit hex code that you will use as a unique identifier in your attacks.

farm.c: The source code of your target’s “gadget farm,” which you will use in generating return-oriented programming attacks.

hex2raw: A utility to generate attack strings.

In the following instructions, we will assume that you have copied the files to a protected local directory, and that you are executing the programs in that local directory.

<h2>2.2        Important Points</h2>

Here is a summary of some important rules regarding valid solutions for this lab. These points will not make much sense when you read this document for the first time. They are presented here as a central reference of rules once you get started.

<ul>

 <li>You must do the assignment on a machine that is similar to the one that generated your targets (Linux, 64-bit, gcc complier)</li>

 <li>Your solutions may not use attacks to circumvent the validation code in the programs. Specifically, any address you incorporate into an attack string for use by a ret instruction should be to one of the following destinations:

  <ul>

   <li>The addresses for functions touch1, touch2, or touch3.</li>

   <li>The address of your injected code</li>

   <li>The address of one of your gadgets from the gadget farm.</li>

  </ul></li>

 <li>You may only construct gadgets from file rtarget with addresses ranging between those for functions start_farm and end_farm.</li>

</ul>

<h1>3          Target Programs</h1>

Both CTARGET and RTARGET read strings from standard input. They do so with the function getbuf defined below:

<ul>

 <li>unsigned getbuf()</li>

 <li>{</li>

 <li>char buf[BUFFER_SIZE];</li>

 <li>Gets(buf);</li>

 <li>return 1;</li>

 <li>}</li>

</ul>

The function Gets is similar to the standard library function gets—it reads a string from standard input (terminated by ‘
’ or end-of-file) and stores it (along with a null terminator) at the specified destination. In this code, you can see that the destination is an array buf, declared as having BUFFER_SIZE bytes. At the time your targets were generated, BUFFER_SIZE was a compile-time constant specific to your version of the programs.

Functions Gets() and gets() have no way to determine whether their destination buffers are large enough to store the string they read. They simply copy sequences of bytes, possibly overrunning the bounds of the storage allocated at the destinations.

If the string typed by the user and read by getbuf is sufficiently short, it is clear that getbuf will return 1, as shown by the following execution examples:

unix&gt; <em>./ctarget</em>

Cookie: 0x1a7dd803

Type string: <em>Keep it short!</em>

No exploit. Getbuf returned 0x1 Normal return

Typically an error occurs if you type a long string:

unix&gt; <em>./ctarget</em>

Cookie: 0x1a7dd803

Type string: <em>This is not a very interesting string, but it has the property …</em>

Ouch!: You caused a segmentation fault!

Better luck next time

(Note that the value of the cookie shown will differ from yours.) Program RTARGET will have the same behavior. As the error message indicates, overrunning the buffer typically causes the program state to be corrupted, leading to a memory access error. Your task is to be more clever with the strings you feed CTARGET and RTARGET so that they do more interesting things. These are called <em>exploit </em>strings.

Both CTARGET and RTARGET take several different command line arguments:

-h: Print list of possible command line arguments

-q: Don’t send results to the grading server

-i FILE: Supply input from a file, rather than from standard input

Your exploit strings will typically contain byte values that do not correspond to the ASCII values for printing characters. The program HEX2RAW will enable you to generate these <em>raw </em>strings. See Appendix A for more information on how to use HEX2RAW.

Important points:

<ul>

 <li>Your exploit string must not contain byte value 0x0a at any intermediate position, since this is the ASCII code for newline (‘
’). When Gets encounters this byte, it will assume you intended to terminate the string.</li>

 <li>HEX2RAW expects two-digit hex values separated by one or more white spaces. So if you want to create a byte with a hex value of 0, you need to write it as 00. To create the word 0xdeadbeef you should pass “ef be ad de” to HEX2RAW (note the reversal required for little-endian byte ordering).</li>

</ul>

When you have correctly solved one of the levels, your target program will automatically send a notification to the grading server. For example:

unix&gt; <em>./hex2raw &lt; ctarget.l2.txt | ./ctarget</em>

Cookie: 0x1a7dd803

Type string:Touch2!: You called touch2(0x1a7dd803)

Valid solution for level 2 with target ctarget PASSED: Sent exploit string to server to be validated.

NICE JOB!

The server will test your exploit string to make sure it really works, and it will update the Attacklab scoreboard page indicating that your userid (listed by your target number for anonymity) has completed this phase.

You can view the scoreboard by pointing your Web browser at

<table width="354">

 <tbody>

  <tr>

   <td width="50">Phase</td>

   <td width="74">Program</td>

   <td width="48">Level</td>

   <td width="61">Method</td>

   <td width="68">Function</td>

   <td width="52">Points</td>

  </tr>

  <tr>

   <td width="50">1</td>

   <td width="74">CTARGET</td>

   <td width="48">1</td>

   <td width="61">CI</td>

   <td width="68">touch1</td>

   <td width="52">10</td>

  </tr>

  <tr>

   <td width="50">2</td>

   <td width="74">CTARGET</td>

   <td width="48">2</td>

   <td width="61">CI</td>

   <td width="68">touch2</td>

   <td width="52">25</td>

  </tr>

  <tr>

   <td width="50">3</td>

   <td width="74">CTARGET</td>

   <td width="48">3</td>

   <td width="61">CI</td>

   <td width="68">touch3</td>

   <td width="52">25</td>

  </tr>

  <tr>

   <td width="50">4</td>

   <td width="74">RTARGET</td>

   <td width="48">2</td>

   <td width="61">ROP</td>

   <td width="68">touch2</td>

   <td width="52">35</td>

  </tr>

  <tr>

   <td width="50">5</td>

   <td width="74">RTARGET</td>

   <td width="48">3</td>

   <td width="61">ROP</td>

   <td width="68">touch3</td>

   <td width="52">5</td>

  </tr>

 </tbody>

</table>

CI:           Code injection

ROP:        Return-oriented programming Figure 1: Summary of attack lab phases

http://cs2400-applied.cs.colorado.edu:15513/scoreboard

Unlike the Bomb Lab, there is no penalty for making mistakes in this lab. Feel free to fire away at CTARGET and RTARGET with any strings you like.

Figure 1 summarizes the five phases of the lab. As can be seen, the first three involve code-injection (CI) attacks on CTARGET, while the last two involve return-oriented-programming (ROP) attacks on RTARGET.

<h1>4          Part I: Code Injection Attacks</h1>

For the first three phases, your exploit strings will attack CTARGET. This program is set up in a way that the stack positions will be consistent from one run to the next and so that data on the stack can be treated as executable code. These features make the program vulnerable to attacks where the exploit strings contain the byte encodings of executable code.

<h2>4.1        Level 1</h2>

For Phase 1, you will not inject new code. Instead, your exploit string will redirect the program to execute an existing procedure.

Function getbuf is called within CTARGET by a function test having the following C code:

<ul>

 <li>void test()</li>

 <li>{</li>

 <li>int val;</li>

 <li>val = getbuf();</li>

 <li>printf(“No exploit. Getbuf returned 0x%x
”, val); 6 }</li>

</ul>

When getbuf executes its return statement (line 5 of getbuf), the program ordinarily resumes execution within function test (at line 5 of this function). We want to change this behavior. Within the file ctarget, there is code for a function touch1 having the following C representation:

<ul>

 <li>void touch1()</li>

 <li>{</li>

 <li>vlevel = 1; /* Part of validation protocol */</li>

 <li>printf(“Touch1!: You called touch1()
”);</li>

 <li>validate(1);</li>

 <li>exit(0);</li>

 <li>}</li>

</ul>

Your task is to get CTARGET to execute the code for touch1 when getbuf executes its return statement, rather than returning to test. Note that your exploit string may also corrupt parts of the stack not directly related to this stage, but this will not cause a problem, since touch1 causes the program to exit directly.

Some Advice:

<ul>

 <li>All the information you need to devise your exploit string for this level can be determined by examining a disassembled version of CTARGET. Use objdump -d to get this dissembled version.</li>

 <li>The idea is to position a byte representation of the starting address for touch1 so that the ret instruction at the end of the code for getbuf will transfer control to touch1.</li>

 <li>Be careful about byte ordering.</li>

 <li>You might want to use GDB to step the program through the last few instructions of getbuf to make sure it is doing the right thing.</li>

 <li>The placement of buf within the stack frame for getbuf depends on the value of compile-time constant BUFFER_SIZE, as well the allocation strategy used by GCC. You will need to examine the disassembled code to determine its position.</li>

</ul>

<h2>4.2        Level 2</h2>

Phase 2 involves injecting a small amount of code as part of your exploit string.

Within the file ctarget there is code for a function touch2 having the following C representation:

<ul>

 <li>void touch2(unsigned val)</li>

 <li>{</li>

 <li>vlevel = 2; /* Part of validation protocol */</li>

 <li>if (val == cookie) {</li>

 <li>printf(“Touch2!: You called touch2(0x%.8x)
”, val);</li>

 <li>validate(2);</li>

 <li>} else {</li>

 <li>printf(“Misfire: You called touch2(0x%.8x)
”, val);</li>

 <li>fail(2);</li>

 <li>}</li>

 <li>exit(0); 12 }</li>

</ul>

Your task is to get CTARGET to execute the code for touch2 rather than returning to test. In this case, however, you must make it appear to touch2 as if you have passed your cookie as its argument.

Some Advice:

<ul>

 <li>You will want to position a byte representation of the address of your injected code in such a way that ret instruction at the end of the code for getbuf will transfer control to it.</li>

 <li>Recall that the first argument to a function is passed in register %rdi.</li>

 <li>Your injected code should set the register to your cookie, and then use a ret instruction to transfer control to the first instruction in touch2.</li>

 <li>Do not attempt to use jmp or call instructions in your exploit code. The encodings of destination addresses for these instructions are difficult to formulate. Use ret instructions for all transfers of control, even when you are not returning from a call.</li>

 <li>See the discussion in Appendix B on how to use tools to generate the byte-level representations of instruction sequences.</li>

</ul>

<h2>4.3        Level 3</h2>

Phase 3 also involves a code injection attack, but passing a string as argument.

Within the file ctarget there is code for functions hexmatch and touch3 having the following C representations:

<ul>

 <li>/* Compare string to hex represention of unsigned value */</li>

 <li>int hexmatch(unsigned val, char *sval)</li>

 <li>{</li>

 <li>char cbuf[110];</li>

 <li>/* Make position of check string unpredictable */</li>

 <li>char *s = cbuf + random() % 100; 7 sprintf(s, “%.8x”, val);</li>

 <li>return strncmp(sval, s, 9) == 0;</li>

 <li>}</li>

</ul>

10

<ul>

 <li>void touch3(char *sval)</li>

 <li>{</li>

 <li>vlevel = 3; /* Part of validation protocol */</li>

 <li>if (hexmatch(cookie, sval)) {</li>

 <li>printf(“Touch3!: You called touch3(”%s”)
”, sval);</li>

 <li>validate(3);</li>

 <li>} else {</li>

 <li>printf(“Misfire: You called touch3(”%s”)
”, sval);</li>

 <li>fail(3);</li>

 <li>}</li>

 <li>exit(0); 22 }</li>

</ul>

Figure 2: Setting up sequence of gadgets for execution. Byte value 0xc3 encodes the ret instruction.

Your task is to get CTARGET to execute the code for touch3 rather than returning to test. You must make it appear to touch3 as if you have passed a string representation of your cookie as its argument.

Some Advice:

<ul>

 <li>You will need to include a string representation of your cookie in your exploit string. The string should consist of the eight hexadecimal digits (ordered from most to least significant) without a leading “0x.”</li>

 <li>Recall that a string is represented in C as a sequence of bytes followed by a byte with value 0. Type “man ascii” on any Linux machine to see the byte representations of the characters you need.</li>

 <li>Your injected code should set register %rdi to the address of this string.</li>

 <li>When functions hexmatch and strncmp are called, they push data onto the stack, overwriting portions of memory that held the buffer used by getbuf. As a result, you will need to be careful where you place the string representation of your cookie.</li>

</ul>

<h1>5          Part II: Return-Oriented Programming</h1>

Performing code-injection attacks on program RTARGET is much more difficult than it is for CTARGET, because it uses two techniques to thwart such attacks:

<ul>

 <li>It uses randomization so that the stack positions differ from one run to another. This makes it impossible to determine where your injected code will be located.</li>

 <li>It marks the section of memory holding the stack as nonexecutable, so even if you could set the program counter to the start of your injected code, the program would fail with a segmentation fault.</li>

</ul>

Fortunately, clever people have devised strategies for getting useful things done in a program by executing existing code, rather than injecting new code. The most general form of this is referred to as <em>return-oriented programming </em>(ROP) [1, 2]. The strategy with ROP is to identify byte sequences within an existing program that consist of one or more instructions followed by the instruction ret. Such a segment is referred to as a <em>gadget</em>. Figure 2 illustrates how the stack can be set up to execute a sequence of <em>n </em>gadgets. In this figure, the stack contains a sequence of gadget addresses. Each gadget consists of a series of instruction bytes, with the final one being 0xc3, encoding the ret instruction. When the program executes a ret instruction starting with this configuration, it will initiate a chain of gadget executions, with the ret instruction at the end of each gadget causing the program to jump to the beginning of the next.

A gadget can make use of code corresponding to assembly-language statements generated by the compiler, especially ones at the ends of functions. In practice, there may be some useful gadgets of this form, but not enough to implement many important operations. For example, it is highly unlikely that a compiled function would have popq %rdi as its last instruction before ret. Fortunately, with a byte-oriented instruction set, such as x86-64, a gadget can often be found by extracting patterns from other parts of the instruction byte sequence.

For example, one version of rtarget contains code generated for the following C function:

void setval_210(unsigned *p) {

*p = 3347663060U;

}

The chances of this function being useful for attacking a system seem pretty slim. But, the disassembled machine code for this function shows an interesting byte sequence:

<table width="518">

 <tbody>

  <tr>

   <td width="319">0000000000400f15 &lt;setval_210&gt;:</td>

   <td width="56"> </td>

   <td width="143"> </td>

  </tr>

  <tr>

   <td width="319">        400f15:                            c7 07 d4 48 89 c7</td>

   <td width="56">movl</td>

   <td width="143">$0xc78948d4,(%rdi)</td>

  </tr>

  <tr>

   <td width="319">        400f1b:                     c3</td>

   <td width="56">retq</td>

   <td width="143"> </td>

  </tr>

 </tbody>

</table>

The byte sequence 48 89 c7 encodes the instruction movq %rax, %rdi. (See Figure 3A for the encodings of useful movq instructions.) This sequence is followed by byte value c3, which encodes the ret instruction. The function starts at address 0x400f15, and the sequence starts on the fourth byte of the function. Thus, this code contains a gadget, having a starting address of 0x400f18, that will copy the 64-bit value in register %rax to register %rdi.

Your code for RTARGET contains a number of functions similar to the setval_210 function shown above in a region we refer to as the <em>gadget farm</em>. Your job will be to identify useful gadgets in the gadget farm and use these to perform attacks similar to those you did in Phases 2 and 3.

Important: The gadget farm is demarcated by functions start_farm and end_farm in your copy of rtarget. Do not attempt to construct gadgets from other portions of the program code.

<h2>5.1        Level 2</h2>

For Phase 4, you will repeat the attack of Phase 2, but do so on program RTARGET using gadgets from your gadget farm. You can construct your solution using gadgets consisting of the following instruction types, and using only the first eight x86-64 registers (%rax–%rdi).

<ol>

 <li>Encodings of movq instructions</li>

</ol>

movq <em>S</em>, <em>D</em>

<table width="690">

 <tbody>

  <tr>

   <td rowspan="2" width="53">Source<em>S</em></td>

   <td width="80"> </td>

   <td width="80"> </td>

   <td width="80"> </td>

   <td colspan="2" width="159">Destination <em>D</em></td>

   <td width="80"> </td>

   <td width="80"> </td>

   <td width="80"> </td>

  </tr>

  <tr>

   <td width="80">%rax</td>

   <td width="80">%rcx</td>

   <td width="80">%rdx</td>

   <td width="80">%rbx</td>

   <td width="80">%rsp</td>

   <td width="80">%rbp</td>

   <td width="80">%rsi</td>

   <td width="80">%rdi</td>

  </tr>

  <tr>

   <td width="53">%rax</td>

   <td width="80">48 89 c0</td>

   <td width="80">48 89 c1</td>

   <td width="80">48 89 c2</td>

   <td width="80">48 89 c3</td>

   <td width="80">48 89 c4</td>

   <td width="80">48 89 c5</td>

   <td width="80">48 89 c6</td>

   <td width="80">48 89 c7</td>

  </tr>

  <tr>

   <td width="53">%rcx</td>

   <td width="80">48 89 c8</td>

   <td width="80">48 89 c9</td>

   <td width="80">48 89 ca</td>

   <td width="80">48 89 cb</td>

   <td width="80">48 89 cc</td>

   <td width="80">48 89 cd</td>

   <td width="80">48 89 ce</td>

   <td width="80">48 89 cf</td>

  </tr>

  <tr>

   <td width="53">%rdx</td>

   <td width="80">48 89 d0</td>

   <td width="80">48 89 d1</td>

   <td width="80">48 89 d2</td>

   <td width="80">48 89 d3</td>

   <td width="80">48 89 d4</td>

   <td width="80">48 89 d5</td>

   <td width="80">48 89 d6</td>

   <td width="80">48 89 d7</td>

  </tr>

  <tr>

   <td width="53">%rbx</td>

   <td width="80">48 89 d8</td>

   <td width="80">48 89 d9</td>

   <td width="80">48 89 da</td>

   <td width="80">48 89 db</td>

   <td width="80">48 89 dc</td>

   <td width="80">48 89 dd</td>

   <td width="80">48 89 de</td>

   <td width="80">48 89 df</td>

  </tr>

  <tr>

   <td width="53">%rsp</td>

   <td width="80">48 89 e0</td>

   <td width="80">48 89 e1</td>

   <td width="80">48 89 e2</td>

   <td width="80">48 89 e3</td>

   <td width="80">48 89 e4</td>

   <td width="80">48 89 e5</td>

   <td width="80">48 89 e6</td>

   <td width="80">48 89 e7</td>

  </tr>

  <tr>

   <td width="53">%rbp</td>

   <td width="80">48 89 e8</td>

   <td width="80">48 89 e9</td>

   <td width="80">48 89 ea</td>

   <td width="80">48 89 eb</td>

   <td width="80">48 89 ec</td>

   <td width="80">48 89 ed</td>

   <td width="80">48 89 ee</td>

   <td width="80">48 89 ef</td>

  </tr>

  <tr>

   <td width="53">%rsi</td>

   <td width="80">48 89 f0</td>

   <td width="80">48 89 f1</td>

   <td width="80">48 89 f2</td>

   <td width="80">48 89 f3</td>

   <td width="80">48 89 f4</td>

   <td width="80">48 89 f5</td>

   <td width="80">48 89 f6</td>

   <td width="80">48 89 f7</td>

  </tr>

  <tr>

   <td width="53">%rdi</td>

   <td width="80">48 89 f8</td>

   <td width="80">48 89 f9</td>

   <td width="80">48 89 fa</td>

   <td width="80">48 89 fb</td>

   <td width="80">48 89 fc</td>

   <td width="80">48 89 fd</td>

   <td width="80">48 89 fe</td>

   <td width="80">48 89 ff</td>

  </tr>

 </tbody>

</table>

<ol>

 <li>Encodings of popq instructions</li>

</ol>

<table width="452">

 <tbody>

  <tr>

   <td rowspan="2" width="69">Operation</td>

   <td width="48"> </td>

   <td width="48"> </td>

   <td width="48"> </td>

   <td colspan="2" width="96">Register <em>R</em></td>

   <td width="48"> </td>

   <td width="48"> </td>

   <td width="48"> </td>

  </tr>

  <tr>

   <td width="48">%rax</td>

   <td width="48">%rcx</td>

   <td width="48">%rdx</td>

   <td width="48">%rbx</td>

   <td width="48">%rsp</td>

   <td width="48">%rbp</td>

   <td width="48">%rsi</td>

   <td width="48">%rdi</td>

  </tr>

  <tr>

   <td width="69">popq <em>R</em></td>

   <td width="48">58</td>

   <td width="48">59</td>

   <td width="48">5a</td>

   <td width="48">5b</td>

   <td width="48">5c</td>

   <td width="48">5d</td>

   <td width="48">5e</td>

   <td width="48">5f</td>

  </tr>

 </tbody>

</table>

<ol>

 <li>Encodings of movl instructions</li>

</ol>

movl <em>S</em>, <em>D</em>

<table width="499">

 <tbody>

  <tr>

   <td rowspan="2" width="53">Source<em>S</em></td>

   <td width="56"> </td>

   <td width="56"> </td>

   <td width="56"> </td>

   <td colspan="2" width="112">Destination <em>D</em></td>

   <td width="56"> </td>

   <td width="56"> </td>

   <td width="56"> </td>

  </tr>

  <tr>

   <td width="56">%eax</td>

   <td width="56">%ecx</td>

   <td width="56">%edx</td>

   <td width="56">%ebx</td>

   <td width="56">%esp</td>

   <td width="56">%ebp</td>

   <td width="56">%esi</td>

   <td width="56">%edi</td>

  </tr>

  <tr>

   <td width="53">%eax</td>

   <td width="56">89 c0</td>

   <td width="56">89 c1</td>

   <td width="56">89 c2</td>

   <td width="56">89 c3</td>

   <td width="56">89 c4</td>

   <td width="56">89 c5</td>

   <td width="56">89 c6</td>

   <td width="56">89 c7</td>

  </tr>

  <tr>

   <td width="53">%ecx</td>

   <td width="56">89 c8</td>

   <td width="56">89 c9</td>

   <td width="56">89 ca</td>

   <td width="56">89 cb</td>

   <td width="56">89 cc</td>

   <td width="56">89 cd</td>

   <td width="56">89 ce</td>

   <td width="56">89 cf</td>

  </tr>

  <tr>

   <td width="53">%edx</td>

   <td width="56">89 d0</td>

   <td width="56">89 d1</td>

   <td width="56">89 d2</td>

   <td width="56">89 d3</td>

   <td width="56">89 d4</td>

   <td width="56">89 d5</td>

   <td width="56">89 d6</td>

   <td width="56">89 d7</td>

  </tr>

  <tr>

   <td width="53">%ebx</td>

   <td width="56">89 d8</td>

   <td width="56">89 d9</td>

   <td width="56">89 da</td>

   <td width="56">89 db</td>

   <td width="56">89 dc</td>

   <td width="56">89 dd</td>

   <td width="56">89 de</td>

   <td width="56">89 df</td>

  </tr>

  <tr>

   <td width="53">%esp</td>

   <td width="56">89 e0</td>

   <td width="56">89 e1</td>

   <td width="56">89 e2</td>

   <td width="56">89 e3</td>

   <td width="56">89 e4</td>

   <td width="56">89 e5</td>

   <td width="56">89 e6</td>

   <td width="56">89 e7</td>

  </tr>

  <tr>

   <td width="53">%ebp</td>

   <td width="56">89 e8</td>

   <td width="56">89 e9</td>

   <td width="56">89 ea</td>

   <td width="56">89 eb</td>

   <td width="56">89 ec</td>

   <td width="56">89 ed</td>

   <td width="56">89 ee</td>

   <td width="56">89 ef</td>

  </tr>

  <tr>

   <td width="53">%esi</td>

   <td width="56">89 f0</td>

   <td width="56">89 f1</td>

   <td width="56">89 f2</td>

   <td width="56">89 f3</td>

   <td width="56">89 f4</td>

   <td width="56">89 f5</td>

   <td width="56">89 f6</td>

   <td width="56">89 f7</td>

  </tr>

  <tr>

   <td width="53">%edi</td>

   <td width="56">89 f8</td>

   <td width="56">89 f9</td>

   <td width="56">89 fa</td>

   <td width="56">89 fb</td>

   <td width="56">89 fc</td>

   <td width="56">89 fd</td>

   <td width="56">89 fe</td>

   <td width="56">89 ff</td>

  </tr>

 </tbody>

</table>

<ol>

 <li>Encodings of 2-byte functional nop instructions</li>

</ol>

<table width="331">

 <tbody>

  <tr>

   <td rowspan="2" width="64">Operat</td>

   <td rowspan="2" width="26">ion</td>

   <td rowspan="2" width="18"> </td>

   <td width="56"> </td>

   <td colspan="2" width="112">Register <em>R</em></td>

   <td width="56"> </td>

  </tr>

  <tr>

   <td width="56">%al</td>

   <td width="56">%cl</td>

   <td width="56">%dl</td>

   <td width="56">%bl</td>

  </tr>

  <tr>

   <td width="64">andb</td>

   <td width="26"><em>R</em>,</td>

   <td width="18"><em>R</em></td>

   <td width="56">20 c0</td>

   <td width="56">20 c9</td>

   <td width="56">20 d2</td>

   <td width="56">20 db</td>

  </tr>

  <tr>

   <td width="64">orb</td>

   <td width="26"><em>R</em>,</td>

   <td width="18"><em>R</em></td>

   <td width="56">08 c0</td>

   <td width="56">08 c9</td>

   <td width="56">08 d2</td>

   <td width="56">08 db</td>

  </tr>

  <tr>

   <td width="64">cmpb</td>

   <td width="26"><em>R</em>,</td>

   <td width="18"><em>R</em></td>

   <td width="56">38 c0</td>

   <td width="56">38 c9</td>

   <td width="56">38 d2</td>

   <td width="56">38 db</td>

  </tr>

  <tr>

   <td width="64">testb</td>

   <td width="26"><em>R</em>,</td>

   <td width="18"><em>R</em></td>

   <td width="56">84 c0</td>

   <td width="56">84 c9</td>

   <td width="56">84 d2</td>

   <td width="56">84 db</td>

  </tr>

 </tbody>

</table>

Figure 3: Byte encodings of instructions. All values are shown in hexadecimal.

movq : The codes for these are shown in Figure 3A. popq : The codes for these are shown in Figure 3B. ret : This instruction is encoded by the single byte 0xc3.

nop : This instruction (pronounced “no op,” which is short for “no operation”) is encoded by the single byte 0x90. Its only effect is to cause the program counter to be incremented by 1.

Some Advice:

<ul>

 <li>All the gadgets you need can be found in the region of the code for rtarget demarcated by the functions start_farm and mid_farm.</li>

 <li>You can do this attack with just two gadgets.</li>

 <li>When a gadget uses a popq instruction, it will pop data from the stack. As a result, your exploit string will contain a combination of gadget addresses and data.</li>

</ul>

<h2>5.2        Level 3</h2>

Before you take on the Phase 5, pause to consider what you have accomplished so far. In Phases 2 and 3, you caused a program to execute machine code of your own design. If CTARGET had been a network server, you could have injected your own code into a distant machine. In Phase 4, you circumvented two of the main devices modern systems use to thwart buffer overflow attacks. Although you did not inject your own code, you were able inject a type of program that operates by stitching together sequences of existing code. You have also gotten 95/100 points for the lab. That’s a good score. If you have other pressing obligations consider stopping right now.

Phase 5 requires you to do an ROP attack on RTARGET to invoke function touch3 with a pointer to a string representation of your cookie. That may not seem significantly more difficult than using an ROP attack to invoke touch2, except that we have made it so. Moreover, Phase 5 counts for only 5 points, which is not a true measure of the effort it will require. Think of it as more an extra credit problem for those who want to go beyond the normal expectations for the course.

To solve Phase 5, you can use gadgets in the region of the code in rtarget demarcated by functions start_farm and end_farm. In addition to the gadgets used in Phase 4, this expanded farm includes the encodings of different movl instructions, as shown in Figure 3C. The byte sequences in this part of the farm also contain 2-byte instructions that serve as <em>functional nops</em>, i.e., they do not change any register or memory values. These include instructions, shown in Figure 3D, such as andb %al,%al, that operate on the low-order bytes of some of the registers but do not change their values.

Some Advice:

<ul>

 <li>You’ll want to review the effect a movl instruction has on the upper 4 bytes of a register, as is described on page 183 of the text.</li>

 <li>The official solution requires eight gadgets (not all of which are unique).</li>

</ul>

Good luck and have fun!

<h1>A         Using HEX2RAW</h1>

HEX2RAW takes as input a <em>hex-formatted </em>string. In this format, each byte value is represented by two hex digits. For example, the string “012345” could be entered in hex format as “30 31 32 33 34 35 00.” (Recall that the ASCII code for decimal digit <em>x </em>is 0x3<em>x</em>, and that the end of a string is indicated by a null byte.)

The hex characters you pass to HEX2RAW should be separated by whitespace (blanks or newlines). We recommend separating different parts of your exploit string with newlines while you’re working on it. HEX2RAW supports C-style block comments, so you can mark off sections of your exploit string. For example:

48 c7 c1 f0 11 40 00 /* mov                                       $0x40011f0,%rcx */

Be sure to leave space around both the starting and ending comment strings (“/*”, “*/”), so that the comments will be properly ignored.

If you generate a hex-formatted exploit string in the file exploit.txt, you can apply the raw string to CTARGET or RTARGET in several different ways:

<ol>

 <li>You can set up a series of pipes to pass the string through HEX2RAW.</li>

</ol>

unix&gt; <em>cat exploit.txt | ./hex2raw | ./ctarget</em>

<ol start="2">

 <li>You can store the raw string in a file and use I/O redirection:</li>

</ol>

unix&gt; <em>./hex2raw &lt; exploit.txt &gt; exploit-raw.txt </em>unix&gt; <em>./ctarget &lt; exploit-raw.txt</em>

This approach can also be used when running from within GDB:

unix&gt; <em>gdb ctarget</em>

(gdb) <em>run &lt; exploit-raw.txt</em>

<ol start="3">

 <li>You can store the raw string in a file and provide the file name as a command-line argument:</li>

</ol>

unix&gt; <em>./hex2raw &lt; exploit.txt &gt; exploit-raw.txt </em>unix&gt; <em>./ctarget -i exploit-raw.txt</em>

This approach also can be used when running from within GDB.

<h1>B       Generating Byte Codes</h1>

Using GCC as an assembler and OBJDUMP as a disassembler makes it convenient to generate the byte codes for instruction sequences. For example, suppose you write a file example.s containing the following assembly code:

# Example of hand-generated assembly code

<table width="438">

 <tbody>

  <tr>

   <td width="64">pushq</td>

   <td width="151">$0xabcdef</td>

   <td width="223"># Push value onto stack</td>

  </tr>

  <tr>

   <td width="64">addq</td>

   <td width="151">$17,%rax</td>

   <td width="223"># Add 17 to %rax</td>

  </tr>

  <tr>

   <td width="64">movl</td>

   <td width="151">%eax,%edx</td>

   <td width="223"># Copy lower 32 bits to %edx</td>

  </tr>

 </tbody>

</table>

The code can contain a mixture of instructions and data. Anything to the right of a ‘#’ character is a comment.

You can now assemble and disassemble this file:

unix&gt; <em>gcc -c example.s </em>unix&gt; <em>objdump -d example.o &gt; example.d</em>

The generated file example.d contains the following:

example.o:                              file format elf64-x86-64

Disassembly of section .text:

0000000000000000 &lt;.text&gt;:

0: 68 ef cd ab 00                                   pushq $0xabcdef

5: 48 83 c0 11                                    add              $0x11,%rax

9: 89 c2                                                 mov             %eax,%edx

The lines at the bottom show the machine code generated from the assembly language instructions. Each line has a hexadecimal number on the left indicating the instruction’s starting address (starting with 0), while the hex digits after the ‘:’ character indicate the byte codes for the instruction. Thus, we can see that the instruction push $0xABCDEF has hex-formatted byte code 68 ef cd ab 00.

From this file, you can get the byte sequence for the code:

68 ef cd ab 00 48 83 c0 11 89 c2

This string can then be passed through HEX2RAW to generate an input string for the target programs.. Alternatively, you can edit example.d to omit extraneous values and to contain C-style comments for readability, yielding:

<table width="319">

 <tbody>

  <tr>

   <td width="135">68 ef cd ab 00</td>

   <td width="183">/* pushq $0xabcdef */</td>

  </tr>

  <tr>

   <td width="135">48 83 c0 11</td>

   <td width="183">/* add                        $0x11,%rax */</td>

  </tr>

  <tr>

   <td width="135">89 c2</td>

   <td width="183">/* mov                       %eax,%edx */</td>

  </tr>

 </tbody>

</table>

This is also a valid input you can pass through HEX2RAW before sending to one of the target programs.

<h1>References</h1>

<ul>

 <li>Roemer, E. Buchanan, H. Shacham, and S. Savage. Return-oriented programming: Systems, languages, and applications. <em>ACM Transactions on Information System Security</em>, 15(1):2:1–2:34, March 2012.</li>

 <li>J. Schwartz, T. Avgerinos, and D. Brumley. Q: Exploit hardening made easy. In <em>USENIX Security Symposium</em>, 2011.</li>

</ul>