
# dbgR:  a Debugging Tool for R
   
<img src = http://heather.cs.ucdavis.edu/debugRcartoon.png>

This tool, **dbgR**, greatly enhances the R debugging process.  R's own
built-in debugging tools are very limited, but **dbgR** extends them to
a rich set of commands, plus visual display of your execution of the
debuggee.  Those who have used the built-in R tools **debug()** and
**browser()** will find many similarities, but no such background is
assumed.  (Note:  RStudio, ESS and StatET all have nice debugging
tools.)

Note that **an additional of goal of this package is to teach good
debugging habits.**  This feature will be developed over time.

## What You Need

* **screen** utility (Mac, Linux)

## Nice to Have

* **xterm** or **gnome-terminal** (the former is installable on Mac, Linux;
  the latter is included in Ubuntu Linux)

* a bit of prior experience with R's **debug()** function

## Quick Start

* After installing and loading **dbgR** type

``` R
dbgR(system.file('examples/test.R',package='dbgR'),NULL)
```

in your R console.  Substitute 'xterm' or 'gnome-term' for NULL if you have it.

That call to **system.file** is just a means of getting the test file
from the **dbgR** packages **examples** directory.  Normally you would
type in your own file to be debugged.

The test file has contents

```R
f <- function() {
   sum <- 0
   sum2 <- 0
   for (i in 1:3) {
      sum <- sum + i
      sum2 <- sum2 + i*i
   }
   c(sum,sum2)
}
```

If you specified a terminal type, the call to **dbgR()** will cause a
new window to be launched, with R running in it.  If you left the
terminal type as NULL, you will be asked to launch a terminal window
yourself by hand; just heed the instructions.  

Either way, let's call this Window 1, and refer to the original one
(from which you called **dbgR**) as Window 0.  During the debugging
process, you will primarily be typing in Window 0, but will use
Window 1 for viewing output of some commands. 

(If you happen to look at Window 1, you'll see that **dbgR** is sending
commands to the R session in Window 1.) 

* In the command area (space at the bottom of Window 0 type)

```
df f
```

to set the function <strong>f()</strong> to debug status.  (Indeed,
you'll see that **debug(f)** was run in Window 1.)

* Then in the command area, type

```
rn f()
```

which instructs the tool to run ("rn") the expression **f()**.  No
arguments in this particular call, but of course you could have some for
other functions.  Any R command can be run here, not just a function
call.

* You can then type 'n' for next line, 'c' for continue, 
'p' to print the value of a variable etc. 
Type 'Q' to quit the browser in Window 1, or 'es' to 
leave the debugging tool.  (Window 1 will go away, and in Window 0, your
original R console will reappear.)

There are many other commands, e.g. conditional breakpoints, automatic
printing of variables/expressions at each pause, etc.  

Type 'h' to see a list of all commands, which will then be displayed in
Window 1, in whatever text editor your R configuration uses.  You can
scroll up and down there, then exit the editor when you're done.

## Command List

Enter key:  repeat last command (should use this a lot, e.g. for
repeating 'n' command)
  
rn expr:  Run the given R expression; if no expression given, then
use the previous Run
  
n,s,c:  go to Next line/Step into function/Continue until next pause
  
df f, udf f:  Debug/Undebug f()
<br>udfa:  Undebug all functions
  
bp linenum:  set Breakpoint at given line
<br>bp linenum expr:  set Breakpoint at given line, conditional on expr
<br>ubp linenum:  cancel Breakpoint at the given line
  
p expr:  Print expression
<br>pap expr:  Print expression at each Pause (only one expression at a time)
<br>upap:  cancel pap
  
pc expr:  Print expression to Console
<br>pcap expr:  Print expression to Console at each Pause 
<br>upcap:  cancel pcap

pls: print local variables (including args) of the current function
<br>penv e: print contents of the environment e

down: scroll down in debugger window
<br>up: scroll up in debugger window
  
Q:  quit R's debugger
<br>es:  exit dbgR program
  
ls srcfile:  (re)load source file; if no file given, use the previous one

## Deleting Old 'screen' Sessions

This package uses the Unix-family **screen** facility.  If **dbgR** ends
prematurely, some **screen** sessions may be left over, and will needed
to be deleted before further use of **dbgR**, which can be done via a
**killscreen()** call

## Known Issues


## How It Works

Consider what happens if we use R's built-in debugging tool on the
above code:

```R
> debug(f)
> f()
debugging in: f()
debug at test.R#2: {
    sum <- 0
    for (i in 1:3) {
        sum <- sum + i
    }
    sum
}
Browse[2]> n
debug at test.R#3: sum <- 0
Browse[2]> 
```

Here we told R to put **f()** in debug status, so when we then executed the
function, we entered R's browser.  Note how at each pause, the browser 
tells us the current line in our buggy source file, lines 2 and 3 in the
example above.  This is used by **dbgR** to update its display of our
source file, as follows.

The R **sink** function (with the arguments used in **dbgR**) copies
all R output to a file.  (For those who know the Unix **tee** command,
this is similar.)  It is then a simple matter for **dbgR** to read the
file, and then update the cursor in the **dbgR** display accordingly.

Now consider the 'n' command in **dbgR**.  What happens when the
user issues that command is that **dbgR** write the string 'n\n' to
the R window.  This is done via the Unix **screen** utility.
