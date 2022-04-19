# Xi must die

```sh
Xi must die
The C language is one of the most influential programming languages ​​in history. It has become an indispensable tool for developing operating systems, displacing assembly languages ​​from this pedestal. Learning C is a must for any self-respecting programmer. This language is loved for its external simplicity and hated for its ruthlessness to errors. Thanks to him, we have the Linux kernel and thousands of vulnerabilities in it to boot. Let's try to understand what is this contradictory C language - a blessing or a curse?

The history of the C language originates in the bowels of the American company Bell Labs and is closely connected with the fate of the UNIX operating system. Its creators, Ken Thompson and Dennis Ritchie, developed their project for the PDP-11 computers, and for the first two years assembly language was their main tool. The laboriousness of writing machine code forced him to look for a replacement, which eventually became the C language. With its help, the kernel of the operating system and most of the utilities were completely rewritten. The C language made it possible to create efficient low-level programs on the PDP-11 with little to no assembly language.

Over time, the question arose of porting UNIX to new hardware platforms. The use of the C language has greatly simplified this task. After all, if only assembly language was used in development, then the operating system would have to be rewritten for each computer architecture. On the other hand, the UNIX sources still contained a lot of code written specifically for the PDP-11 computer. And the C language itself did not always accurately reflect the features and details of a particular hardware platform. The latter made the porting process even more difficult and deprived the language of one of its main advantages - transparent and understandable generation of machine code. The more computer architectures C took over, the less obvious its connection to the low level became.

In the process of migrating UNIX to new hardware platforms, another problem was discovered. Ported C programs ran slower than expected. The more the target computer architecture differed from the PDP-11, the less efficient the resulting code was. To compensate for this shortcoming, compiler developers increasingly began to use implicit optimizations. And although this solution improved the performance of the programs themselves, C was moving further and further away from the low level. Now it was necessary not only to understand exactly how the language constructs for each of the computer architectures were defined, but also how they were optimized. Of course, any compiler independently decided how to translate the source code for each hardware platform. As a result, write a low-level program in C language,

It was necessary to understand how to efficiently implement the high-level constructs of the C language, while retaining its low-level properties. An attempt to solve this problem was the publication in 1989 of the first language standard. It is commonly referred to as "ANSI C" or "C89" and is what we will refer to in the following. The creators of the standard decided to finally break the link between C and the PDP-11 architecture and make the language completely high-level. The so-called "abstract machine" was introduced - an imaginary executor of C code (section 2.1.2.3, "Program execution"):

The semantic descriptions in this Standard describe the behavior of an abstract machine in which optimization questions do not matter.
This means that compiler optimizations will not affect the operation of the program as long as its source code conforms to the standard. The abstract machine had to solve two problems at the same time. First, following the standard made it possible to create easily portable C programs. Second, the abstract machine could give compilers the freedom to optimize. But a quite reasonable question arises - how then does the C language differ from any other high-level compiled language? The answer lies in the text of the standard. To still give programmers a theoretical opportunity to write low-level procedures, and therefore non-portable, one more concept was introduced - undefined behavior (undefined behavior, section 1.6, "DEFINITIONS OF TERMS"):

Undefined behavior - behavior when using a non-portable or erroneous programming construct, erroneous data or objects with undefined values ​​for which the standard does not impose any requirements. Possible undefined behavior ranges from completely ignoring the situation with unpredictable results, behavior during translation or program execution in a documented manner specific to the environment (with or without issuing a diagnostic message), to aborting translation or execution (with issuing a diagnostic message).
Simply put, undefined behavior is a deliberately left hole in the description of the C abstract machine. They allow compilers to decide for themselves how to deal with those or other language constructs, the behavior of which the text of the standard is deliberately silent about. Including they can be perceived as invalid in the text of the program. Let's take a closer look at undefined behavior using a specific example.

Let's take the following code snippet in C language:

int x = 1;
x = x << sizeof(int) * 8;
Let's try to guess what the result will be. Let's say we compiled this code for ARM architecture processors. The bit shift instruction within this hardware platform is defined so that the final value of the variable "x" must be "0". On the other hand, we can translate our program into x86 machine code. And already there, the bit shift is implemented in such a way that the value of "x" does not change and remains equal to "1". We could conclude that the result of this piece of code depends on the hardware platform we compiled it for. But actually it is not.

In fact, this piece of code can be processed by the compiler in any possible or impossible way. The reason is as follows: according to the text of the C language standard, a bit shift by an amount greater than or equal to the size of the expression in bits is undefined behavior. It turns out that there is no guarantee that this piece of code will work at all. In fact, even within the same architecture, the same compiler can generate completely different executables. Let's give examples of compiling and running the program with printing the value of the variable "x". In both cases, we use the gcc compiler version 10.2.1 for the x86-64 target architecture.

$ cat test.c
#include <stdio.h>

int main()
{
    int x = 1;
    x = x << sizeof(int) * 8;
    printf("%d\n", x);
    return 0;
}
$ gcc test.c -o test
$ ./test
one
$ gcc -O test.c -o test
$ ./test
0
The "-O" flag allows the gcc compiler to use source code optimizations. Which optimization mechanisms can be applied, as well as which flags are responsible for them, depends on the specific compiler. In general, it is impossible to know how undefined behavior will be handled in a program when the source code is translated. Therefore, the only way to write portable C programs is to completely avoid undefined behavior during development.

Let's look at a slightly more complex example. Another type of undefined behavior is null pointer dereference. Its trivial variant would be the following code snippet:

* (char *) 0;
Of course, no one in their right mind would write something like this in their program. However, it is not necessary to explicitly dereference a null pointer to cause undefined behavior. The "What Every C Programmer Should Know About Undefined Behavior" article series on blog.llvm.org provides a code snippet to prove this:

void contains_null_check(int *p)
{
    int dead = *p;
    if(p == 0)
        return;
    *p = 4;
}
The example may seem far-fetched, but it gives a little more insight into how the C compiler works. The latter uses various optimization mechanisms, but here we are only interested in two. One of them removes redundant, "dead" code, and the second removes useless checks for a null pointer. If the first optimization mechanism is applied to the above code fragment, then it will transform the function as follows:

void contains_null_check(int *p)
{
    if(p == 0)
        return;
    *p = 4;
}
Then the second mechanism will not detect unnecessary checks for a null pointer, and the source code of the function will take its final form. However, in reality, the order of optimizations may be different. For example, the compiler has the right to first of all exclude unnecessary checks for a null pointer, and then the function will be transformed as follows:

void contains_null_check(int *p)
{
    int dead = *p;
    if(0)
        return;
    *p = 4;
}
Since we dereference the pointer before checking it, the compiler safely assumes that the pointer itself will never be null. This replaces the "p == 0" comparison with an expression that always returns false. The compiler then runs the first optimization mechanism and cleans up the "dead" code:

void contains_null_check(int *p)
{
    *p = 4;
}
It is important to emphasize that both of these optimizations are correct. Removing a check can be an unexpected gift from the compiler developers to an inattentive programmer. Such code can create a vulnerability for programs running without memory protection, i.e. operating system kernels or microcontroller firmware. Of course, this example contains an error, but the main problem is not in it, but in how the compiler processes it.

Suppose you accidentally allowed undefined behavior in your program. In the best case, you will immediately find the error and fix it. In not so successful - do it not immediately. However, the situation is much more likely that the compiler will not use your oversight to its advantage. In this case, undefined behavior will remain in the source code of the program until it is declared at the most inopportune moment. And such a moment can come when you change: the target computer architecture, the compiler or even its version, optimization flags, or any other flags in general. Simply put, undefined behavior is a ticking time bomb. When it explodes is not clear, but one can only guess how many interesting surprises the source codes of thousands of programs store in themselves.

Compiler optimizations can also affect functions in the C standard library, including, for example, memset. It is widely and infamously known for the abundance of errors that programmers make when calling it. The function header looks like this:

void *memset(void *ptr, int value, size_t num);
memset writes "num" bytes with value "value" to address "ptr". Even though the "value" parameter is of type int, only its low byte is actually used. The function is actively used to reset large data arrays, however, the compiler itself often likes to insert its call where it is needed and not so much. So, a curious case was discussed on April 15, 2018 on the osdev.org forum. A user named ScropTheOSAdventurer created a topic in which he talked about the process of developing his own educational operating system. To his misfortune, he allowed the compiler to optimize the source code of the project, as a result of which the latter stopped working. While debugging, the programmer found an error in the following code snippet:

void *memset(void *ptr, int value, size_t num)
{
    unsigned char *ptr_byte = (unsigned char *) ptr;
    for(size_t i = 0; i < num; ptr_byte[i] = (unsigned char) value, i++);
    return ptr;
}
For his operating system, the developer decided to use his own implementation of the memset function. But he did not take into account that during the translation process, the gcc compiler will find a very tempting opportunity for optimization in this code. In fact, the function was eventually converted to the following form:

void *memset(void *ptr, int value, size_t num)
{
    return memset(ptr, value, num);
}
It is likely that there were consummate sophistries among the developers of the gcc compiler. In any case, the compiler's ability to optimize clearly exceeds all limits available to the human mind.

Let's take another example with the memset function. The compiler is able not only to create its calls from scratch, but also to throw them out of the source code at will. Thus, in cryptographic programs it is often useful to erase all data from memory after they are no longer needed. Usually this behavior is redundant, but imagine the following situation. Your program works with a database of users that stores usernames and passwords, and you have described a function like this:

int check_password(const char *pwd)
{
    char real_pwd[32];
    get_password(real_pwd);
    return !strcmp(pwd, real_pwd);
}
There is only one problem - after calling check_password, a string with the user's real password will remain on the stack. If your program has at least one vulnerability that allows reading data from memory, then there is a real possibility of stealing the password from the stack. An example of such a vulnerability is the infamous " Heartbleed " bug. To reduce possible risks, the easiest way is to clear the stack fragment containing the password:

int check_password(const char *pwd)
{
    int result;
    char real_pwd[32];
    get_password(real_pwd);
    result = !strcmp(pwd, real_pwd);
    memset(real_pwd, 0, sizeof(real_pwd));
    return result;
}
It would seem that a solution has been found, but not everything is so simple. A compiler versed in optimization issues can decide that the memset call is superfluous here, and calmly remove it from the function body. Indeed, for the operation of the program itself, this action is absolutely useless. Even worse, the compiler can generate code that ends up with the password in one of the processor's registers. In this case, it may be even easier to obtain it using a vulnerability in the program. And you can't even get the compiler to clear the contents of the register. You can read more about this issue at the link .

One of the most insidious varieties of undefined behavior is strict aliasing. The term can be translated as "strict imposition", but it does not have a traditional name in Russian. For this reason, we will use the original English term. The text of the standard gives the following description for strict aliasing (Section 3.3, "EXPRESSIONS"):

The value of an object must be accessible only by an lvalue expression of one of the following types:
- the declared type of the object,
- the qualified version of the declared type of the object,
- the signed or unsigned type that corresponds to the declared type of the object,
- the signed or unsigned type that corresponds to the qualified version of the declared type object,
- an array, structure, or union type that includes one of the above types among its members (including, recursively, a member of an internal structure, array, or union),
- character type.
The easiest way to illustrate strict aliasing is with a specific example:

int x = 42;
float *p = &x;
*p = 13;
To cause undefined behavior, it is enough to refer to any variable of a type that is incompatible with the declared one. This restriction can be circumvented by using a character type (char), which is not subject to strict aliasing rules:

int x = 42;
char*p = &x;
*p = 13;
That's just the separation of the variable into characters can be a laborious task. You will have to consider the size of the data as well as the endianness used. You can also avoid undefined behavior using unions:

union u { int a; short b};
union ux;
xa = 42;
xb = 13;
However, this method is not without drawbacks - the union must contain members with all possible types that will be used by the program. All this seriously complicates the use of "type punning" or the so-called typing pun - intentional violation of the type system. This technique is necessary for more flexible low-level memory management of the machine.

To illustrate the usefulness of the typing pun, let's look at a small example. Let's say you read the contents of an image file into the program's memory. And now you need to write a function that returns the color of a pixel at the specified coordinates. For simplicity, we will assume that the size of the int type is the same as the pixel size, as is the byte order for both:

int get_pixel(const char *buf, int width, int x, int y)
{
    buf += get_header_size(buf);
    return ((const int *) buf)[y * width + x];
}
When the function is called, it is passed the address of a data area containing the contents of the file, including its header, the width of the image, and the coordinates of the pixel whose color should be returned. Instead of the int type, we could have chosen any other type with a known size. But none of this matters, because the get_pixel function is completely wrong from the point of view of the standard, as it violates the rules of strict aliasing. To use the typing pun would require rewriting all the code associated with the buffer being used, including the one responsible for reading the file.

There are a huge number of examples of programs that do not satisfy the rules of strict aliasing. These include the famous fast inverse square root function from Quake 3:

float FastInvSqrt(float x)
{
    float xhalf = 0.5f*x;
    int i = *(int *) &x;
    i = 0x5f3759df - (i >> 1); /* What the fuck? */
    x = *(float *) &i;
    x = x * (1.5f - (xhalf * x * x));
    return x;
}
This code made it possible to calculate the inverse square root of a floating point number several times faster than using an arithmetic coprocessor. However, even this masterpiece of black magic does not pass the test of the standard - it seems that the creator of cult games, John Carmack, does not understand the C language at all.

One question remains - why is this strict aliasing needed at all? The thing is that it allows compiler creators to apply extremely aggressive source code optimizations. Strict aliasing rules apply to access to any memory, including dynamic memory. So the standards committee noted that the following code snippet ( source ):

void f(int *x, double *y)
{
    *x = 0;
    *y = 3.14;
    *x = *x + 2;
}
can be converted like this:

void f(int *x, double *y)
{
    *x = 0;
    *y = 3.14;
    *x = 2;
}
According to the rules of strict aliasing, the y pointer cannot contain the address of the same memory location as the x pointer. It is this fact that allows us to replace the expression "*x = *x + 2" with "*x = 2". The active use of such optimizations by compilers broke a huge amount of old code. So, in a letter dated July 12, 1998, one of the developers of the gcc compiler Jeff Law, answering questions about strict aliasing and related errors, writes ( source ):

There is a lot of code that violates the rules of strict aliasing. One such example is the "portable" generic IP checksum function found in the BSD source codes for networking.
IMHO, there is less and less such code - modern compilers have been using strict aliasing in their analysis for some time, as a result of which people have been forced to correct their code. Of course, this does not apply to Linux and some other free projects, since they only use gcc.

If we start talking about how such code is wrong, then we'd better have some kind of plan in case people start asking why their code, which worked for many years, is now broken.
Point them to the C standard :-) :-)

The strict aliasing rules for the gcc compiler can be enabled using the "-fstrict-aliasing" flag and disabled using the "-fno-strict-aliasing" flag. The latter is recommended if you are not sure if you are violating the text of the standard - most likely you are. Speaking about the Linux kernel mentioned in the letter, its author Linus Torvalds also gave his assessment of strict aliasing in particular and the work of the committee in general. So, criticizing the desire of one of the developers of the operating system to once again play it safe from violating the standard, Linus wrote the following letter ( source ):

Frankly, all this seems doubtful to me.

And I'm not talking about the changes themselves - I can put up with that. But the justification for these very changes is absolute and complete nonsense, and very dangerous at that.

The thing is, using unions to implement a typing pun is the usual and STANDARD way to do it. It's actually documented for gcc, and is used when you're not too smart (original: "f*cking moron"), you've applied "-fstrict aliasing" and now you need to get rid of all the damage that imposes this garbage standard.

Andy, what was the reason for all this idiocy? And don't tell me that the text of the standard is "not clear enough". The standard text is clearly bullshit (see above for strict aliasing rules) and should be ignored in such cases. To do this, you must use compiler tools to avoid damage. Similarly, you need to do in situations where there is no complete clarity.

This is why we use "-fwrapv", "-fno-strict-aliasing" and other flags.

I have already said this before and I will repeat it again: when the text of the standard contradicts reality, it is an ordinary piece of toilet paper. It has absolutely no importance. In fact, I'd rather take a roll of real toilet paper instead - so at least I don't get ink on my ass (original: "won't have splinters and ink up my arse").

Apparently, Linus Torvalds did not learn the C language well - a real C programmer would not have thought of such a thing.

However, the standard is not complete with strict aliasing alone. You don't even need to dereference the pointer to invoke undefined behavior:

void f(int *p, int *q)
{
    free(p);
    if(p == q) /* Undefined behavior! */
        do_something();
}
Using a pointer value after the memory has been freed is prohibited by the text of the standard (Section 4.10.3, "Memory management functions"):

The value of a pointer pointing to freed memory is undefined.
It is important for a programmer to understand that pointers in C are not low-level. The standard tried to completely eradicate any connection between the language and the real world. Even comparing pointers that refer to different objects is declared undefined behavior (Section 3.3.8, "Relational operators"):

When comparing two pointers, the result depends on the relative location in the address space of the objects they point to. If the specified objects are not members of the same structure, array, or union, then the result is undefined.
Here is a small piece of code that demonstrates a comparison that is incorrect from the point of view of the standard:

int *p = malloc(64 * sizeof(int));
int *q = malloc(64 * sizeof(int));
if(p < q) /* Undefined behaviour! */
    do_something();
However, the most interesting example here is the source code for the following program:

#include <stdio.h>

int main()
{
    intx;
    int y;
    int *p = &x + 1;
    int *q = &y;
    printf("%p %p %d\n", (void *) p, (void *) q, p == q);
    return 0;
}
If you translate the above text with the gcc compiler, passing it the "-O" flag, then the resulting executable file, when run, will produce something like the following line:

0x1badc0de 0x1badc0de 0
It remains a big mystery why two pointers containing the same values ​​were not equal. Perhaps the developers of the gcc compiler we have already noted are no less skillful in interpreting the texts of the standard than in matters of sophistry. You can feel the subtle hermeneutics in the discussion of this issue on the official website of the GNU organization.

Most of the pointer examples come from kristerw.blogspot.com. On it you can find more information about the texts of the C standard, as well as arcane compiler optimizations.

It may seem that in the case of disabled optimizations, all the above problems will bypass you. Just don't pass the "-O" flag to the compiler and you'll get the result you expect. But actually it is not. In January 2007, on the site gcc.gnu.org, a user under the nickname felix-gcc posted the source code for the following program:

#include <assert.h>

int foo(int a) {
  assert(a + 100 > a);
  printf("%d %d\n",a + 100, a);
  return a;
}

int main() {
  foo(100);
  foo(0x7fffffff);
}
The foo function checks for overflow the sum of the supplied signed number and the constant "100". As you know, on the vast majority of computer architectures, negative numbers are given in the form of an additional code. In case of overflow, such a number reverses its sign, so that the test "a + 100 > a" returns false. In the body of the main function, felix-gcc calls foo twice. First, it gives the input a number that will not lead to overflow. Then, assuming the size of the int data type is four bytes, felix-gcc calls foo with the largest positive number of that type. It is logical to assume that in this case the comparison will return false, and assert will abort the program. However, here is the output felix-gcc got after running the executable:

200 100
-2147483549 2147483647
In fact, gcc decided to remove the overflow check, and this despite the fact that no flags were passed to the compiler. And even more interesting, earlier versions of gcc did not remove the check under the same conditions, resulting in the resulting program behaving differently. A reasonable request from felix-gcc to fix an unexpected compiler bug was answered by a user named Andrew Pinski. As a developer of gcc, Andrew Pinski noticed that this behavior is not erroneous. Moreover, he himself turned out to be the author of the change in the compiler code, which created such a strange result. The following is a snippet of a conversation between felix-gcc and Andrew Pinski. Comments are redundant:

Andrew Pinski Signed
number overflow is undefined behavior in the text of the C language standard, use the unsigned type or the "-fwrapv" flag.

felix-gcc
You must be joking?
Various security issues are caused by number overflows and you're just telling me that in gcc 4.1 I can no longer test them for signed types? You obviously don't understand something. There MUST be a way around this problem. Existing software uses signed numbers and I can't just change the type to unsigned - I still need to check for overflow! It doesn't look like I could use any workaround for this. What do you want from me - that I cast the type to unsigned, shift to the right by one, then add or whatever?!
PLEASE REVOKE THIS CHANGE. It will create SERIOUS SECURITY ISSUES in ALL POSSIBLE PROGRAMS. I don't care what your standardizers say that gcc is good. ALL OF THIS WILL LEAD TO PEOPLE HACKING. I discovered this issue because one of the security checks that prevents the hack failed.
IT IS NOT JOKE. FIX IT! RIGHT NOW!

Andrew Pinski
I'm not kidding, the C standard explicitly says that signed overflow is undefined behavior.

felix-gcc
So listen Andrew, do you really think this problem will go away if you keep fixing bugs fast enough? The check I wrote covered all possible situations. It is not even necessary to specify what kind of type is used - a pointer, unsigned or signed number. Well, you broke the pointers too, but your changes were fixed. The guy who did it then needs to show up here, we need someone with a clear head and a vision like his.
Let's face it - you screwed up completely (original: "fucked up this royally"), and now you are trying to cover up all the mistakes as soon as possible so that no one notices how much damage you have caused. You sir are unprofessional and a disgrace to the gcc development team. This bug will remain open until you revert everything back or make the flag you mentioned the default. As long as you break programs whose authors foolishly turned on optimizations, I don't care. But I won't let you make my environment less secure just because your unprofessionalism keeps you from dealing with optimizations after they've been shown to do more harm than good. How much more proof do you need to provide? Oh my god yes autoconf thinks your "optimizations" should be turned off everywhere. Do you notice explosions around you at all?

Andrew Pinski

int chkadd(int a, int b)
{
    if(b < 0)
        return chksub(a,-b);
    else
    if(INT_MAX - b < a)
    {
        fputs("int overflow\n", stderr);
        return INT_MAX;
    }
    return a + b;
}
That's all I'm going to refer to from now on. This code tells you clearly how to recognize an overflow before it happens. Again, your code is broken and not up to standard.
felix-gcc
MY CODE IS NOT BROKEN.
Attempts to devalue the problem or insult me ​​will not solve anything.

Andrew Pinski
You wrote a bug, which is why I thought your code was broken.

felix-gcc
So tell me what part of my argument you don't understand? I could use simpler words so you can understand me this time.
Guys, your task is not just to implement the C standard. You also have an obligation not to disrupt programs that depend on you. A LOT of programs depend on you. When you break floating point precision, you make it available with the ("-ffast-math") flag. When you added strict aliasing, you also made this feature available via the ("-fstrict-aliasing") flag. If I remember correctly, then you also quoted the text of the standard, until people with a more adequate understanding of the world stopped you. And I'm going to leave this error open until the same story repeats itself.

Andrew Pinski
I don't think we should have made this optional, but I wasn't there when the decision was made. Also remember that we had a release where strict aliasing was enabled, but then we had to disable it by default. During this time, people corrected their programs while the optimization was active. And we've already made optimizing signed overflow optional with "-fwrapv". I don't understand what your arguments are.

felix-gcc
You can't just potentially break a bunch of free software just because you change your mind about how much freedom the standard gives you.
Grow up or leave, letting more responsible people handle your business.

Andrew Pinski
Wait, but this optimization has been around since 1994, and if any code since that time has used signed overflow, then the authors of these programs have asked for it.

felix-gcc
Did you know that the Ariane-5 rocket exploded (and could have killed people!) due to an overflow error? What if people die because you think the standard allows you to throw out human-written security checks?

Andrew Pinski
I've already shown you how to check for a signed overflow before it happens, not after. You can teach other security people how to write this code.

felix-gcc
Once again: IT DOESN'T MATTER WHAT THE STANDARD SAYS. You broke programs and people got hurt. Now put everything back. The least you can do is make "-fwrapv" the default. You still have to get it to work correctly (I've heard it doesn't work correctly in certain situations), but that's another story.

Andrew Pinski
It will be the default in languages ​​where this behavior is defined. I've given you a way to write overflow checks, and if you don't like what the C standard says, it's not my fault.
Remember: the gcc compiler is also an optimizing compiler, and if you need optimizations, then you must follow the rules of the language you are writing in, instead of creating incorrect programs, which is what happens with C and C++ in general.

felix-gcc
In earlier versions of the compiler, this behavior only happened when optimizations were enabled. If you take a closer look, it turns out that all your arguments are worthless.
Because gcc 4.1 throws out this code already without optimizations enabled. That's all your arguments.
Please make "-fwrapv" the default and I'll shut up.

Andrew Pinski
Try checking the program execution time with and without "-fwrapv". You will see that the code runs faster without it.

Trying to get into the discussion of compiler optimizations, Andrew Pinski decided to justify his position. In the process, however, he mentioned a much more "interesting" argument:

The fact that a person wrote an overflow check the wrong way is not grounds for punishing people who actually did it right using the method described in the documentation. This is my position - you are trying to punish people who wrote their checks in the way that the C language standard suggests.
And at the very end of the discussion, Andew Pinski stated the following:

I'd accept your idea of ​​including "-fwrapv" by default if there wasn't a way to check for an overflow before it happened, but there is. Yes, we will break code that was written on the assumption that signed overflow is possible. But I think this is the price we can accept.
In conclusion, I would like to quote one more quote from Linus Torvalds:

The developers of gcc are more interested in trying to figure out what else the standard allows them to do than in how to make things actually work.
And this seems to be the main problem with the C language. But this could not happen in a vacuum - in the end, we ourselves allowed it to happen. The C language has long ceased to fulfill the functions assigned to it and has become an ugly parody of itself. But we did not notice this, because we resigned ourselves to the fact that our programs do not work. We, as programmers, are so accustomed to errors that they have become an integral part of our lives. Often, debugging and testing programs takes more time than designing and writing the actual code. And it's no wonder - people tend to make mistakes. Most of the bugs and vulnerabilities are introduced by programmers by accident, without thinking at all, and there is nothing we can do about it. However, the inevitability of errors does not justify their existence. The job of a programmer is to write code which is working. Even if it is not obvious, difficult and impossible, we have no right to make mistakes. Because otherwise everything is meaningless, and we cease to understand what can and cannot be done, what is beautiful and what is ugly. In the pursuit of efficiency, compiler developers have forgotten what C is really for. He is a programmer's tool, and you can't write a good program with a bad tool. This story is an illustrative example of the fact that not every activity is fruitful, and not every change leads to a better result. Through the efforts of the standards committee and compiler developers, we eventually lost the C language. As a development tool, it has become completely useless and even harmful, and we must admit this. Otherwise, our programs will never work.

PS If you still believe that the C language can be saved, check out the link below for the following excerpt by one of the two editors of the C standard text:

We will let the compiler lie to you. We will lie to your code. And when things go wrong - a bug, a bugger, a memory leak - we solemnly shake our heads.
The authors express their gratitude to Andrei Viktorovich Stolyarov for his criticism and comments, thanks to which this article has acquired its present form, and without which, perhaps, it would not exist at all.

Nikita Orlov exegete
Alexey Veresov aversey

```
