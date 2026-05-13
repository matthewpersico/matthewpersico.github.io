---
topic: Tech
---
# My Journey with Devel::ptkdb

## Origins

There are a number of well-known schisms in the world of computing:
- Emacs vs Vi
- Tabs vs Spaces
- Perl vs Python
- CLI vs GUI
- Windows vs everything else

I'd like to add another one: Debuggers vs Print Statements.

I come down on the side of debuggers. I'm a visual person and I'm impatient. I don't like to wait for results. I like to get in there, crawl around, change things, see what's going on.

I first started using debuggers on [Digital Equipment Corporation](https://en.wikipedia.org/wiki/Digital_Equipment_Corporation) [VAX](https://en.wikipedia.org/wiki/VAX) machines running [VMS](https://en.wikipedia.org/wiki/OpenVMS). I moved to Unix and used [gdb](https://en.wikipedia.org/wiki/GNU_Debugger) in [Emacs](https://www.gnu.org/software/emacs) and then found [ddd](https://www.gnu.org/software/ddd/).  Any time I started with a new language, one of the first things I reached for was a visual debugger.

When I started using Perl in the waning years of the last century, I conducted my usual search and found [Devel::ptkdb](https://metacpan.org/pod/Devel::ptkdb). It served me well for all the years in which my primary language was Perl. Now, it did have its quirks. The variable display was less than ideal; there was no way to sort the variables in the vertically oriented display list. If you happened to have an array displayed before other variables, then as the array expanded, the other variables would scroll beyond the bottom of the window. The map of loaded modules sometimes did not properly find the related code. The map's sort order was alphabetical and did not bring the most common modules, nor the local modules, to the top of the list where it would be more convenient to access them.

Although I might have tried to fix some of these issues, I was intimidated by the fact that this was a debugger, that it was a rather large piece of complicated software that interfaced with the command line Perl debugger. For what it did, it made my programming life much easier, and I was able to work around the quirks.

That all ended in 2009 when I changed jobs. Perl was no longer my primary environment. I ended up using a proprietary language written by a particular financial institution. There, I was not so intimidated. I enhanced that language's debugger by adding a screen to manage breakpoints.

My next job was at a Java shop where I naturally learned how to use the Java debugger in what I think was the Eclipse ecosystem. At this shop, I was given a task to work on an old Perl program. The program read and wrote various files, but the file names were hard coded. They wanted me to make those names variable. In the process of making this change, I had to work on a production system because there was no dedicated development system. In order to make sure that I didn't disturb production data, I was put on a read-only file system.

So now I had a chance to go back and work with Perl, and by extension, `Devel::ptkdb`. Once I had made my changes, I stepped through the program in the debugger in order to check them. Along the way, I set breakpoints and added variables in the variable display. `Devel::ptkdb` allows you to save the breakpoints and the variable list in a `.ptkdb` state file, which I did.

Imagine my surprise when I started up the program again in the debugger and my saved state was not loaded. I scratched my head for about 10 seconds and realized, "Ah, I'm on a read-only partition and the default location of the state file is the directory where the Perl script lives. That location is a read-only file system. Let me save the state file in my home directory." [1](#footnote-1)<a id="footnote-1-return"></a>

Imagine my surprise when I started up the program again and my saved state did not reappear again. I scratched my head for a bit more than 10 seconds. I tried it again. It was still not saved. After about half a day of having to reset variables and breakpoints every time I ran the program, I decided that it was high time I took a look at the debugger code. I figured that it had to be something simple, related to just the input box for the name of the state file; it was not even a proper file dialog. I was sure I could handle that.

I was able to find the issue and correct it. By using the local corrected version of the debugger, I was able to save my state file to any writable directory.

It was at that point that something clicked in my head. If I could make that fix, maybe I could make other fixes. Maybe I could fix some of the quirks that bothered me all those years ago.

About two months and a huge changelog later, I had fixed some of those quirks. I fixed the sort problem for the variables. I fixed the mapping problem to the loaded code. I even made some improvements. The debugger used to not reload the state file if you restarted the debugger. Now it did. I programmed in a separate widget that I could use to print messages from inside the debugger while I was debugging the debugger.

I then realized I probably shouldn't keep this to myself, so I contacted the author, figuring that they would like to take these changes and update the code. This was approximately 2013, and the debugger hadn't been touched since 1999. However, it turned out that the author had moved on and would prefer that I just took over the project. I agreed, but before I could talk to legal about trying to let this code out of the shop, I ended up leaving that job.

I left all these changes behind a corporate firewall.

And that's where they stayed. Over the years, I had occasion to use the debugger again, but never quite had the 'tuits to recreate all these changes.

Well, now I have the 'tuits. I'm currently unemployed and I have committed to making a presentation at the next [Perl and Raku Conference](https://tprc.us/tprc-2026-gsp/) about the debugger. It's time to roll up my sleeves, dig in and see if I cannot recreate what I did over ten years ago.

##### Footnotes
1. <a id="footnote-1"></a>
Which, all these years later begs the question, "If the file system was read only, how did you update the script you were supposed to be changing?" Let's not let details get in the way of a good story, shall we? [Back](#footnote-1-return)
