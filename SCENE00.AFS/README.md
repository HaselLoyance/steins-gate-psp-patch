# SCENE00.AFS
`SCENE00.AFS` container contains BIN files, which are the game scripts. This is the most important part of the translation, since each BIN file contains the strings of the story text.

For the translation purposes I had to come up with a toolset that would allow me to easily unpack, edit and repack BIN files. While usually people use more sophisticated method for this *\*cough-cough\** Cartographer *\*cough-cough\** the topographer was born to mimic the same functionality and more.

I'm actually very proud of making/discovering this, so here goes explanation of technical details.

**Technical information:**

The best way to represent the structure of script files is to think about each BIN file in two sections. *Section 1*: CriWare Engine byte code, and *Section 2*: string data. In all BIN files there *Section 1* ends in a sequence of two bytes `0x0D 0x02`, and *Section 2* begins immediately after that sequence. I usually refer to the address where *Section 2* begins as *splitAddress* (the address of the first character of the first string).

**NOTE**: Some BIN files might not have *Section 2*. For this I would recommend you to manually pick which files you want to use with topographer, as I cannot guarantee that it won't break on files without *Section 2*. (and really, if there is no *Section 2* with game strings, then why would you even use topographer?)

Every string in *Section 2* is null-terminated (ends in 0x00). This way, it is possible to go through *Section 2* and get the address of the beginning of each game string. For each of the addresses we switch the endian.

Suppose *splitAddress* is `0x00008120`, after endian switch it becomes 0x20810000, which is our *lookupSequence* to look for in the *Section 1*.

What we do next is we search for the lookupSequence in Section 1 (in this case, we look for only two consecutive bytes 0x20 and 0x81, zeroes don't have to match). Once we find it, we take the address of where we found it, pointerAddress, and that's what goes into Atlas scripts inside of #WRITE(...).
The other thing to note is that in the example above we only look for the first 16 bits of the lookupSequence. This is correct, but it can also be 32 bits, if the file is longer. Lets assume that at some point in the BIN file there is beginning of the string at 0x00812340. Then reversing it gives us the lookupSequence of 0x40238100. And we look for exactly that, zero included. The maximum length of the lookupSequence in bits is what goes into #CREATEPTR(PTR, "LINEAR", 0, XX).
Also important to do all the searching in Section 1 with the pointerAddress offset just so that there are no conflicts with the previous data, also it slightly optimizes it
And that's pretty much how the parsing works.
