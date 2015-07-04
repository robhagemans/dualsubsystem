--------
## Dualsubsystem ##
This is a copy of dualsubsystem, originally published under the [MIT license](http://www.opensource.org/licenses/mit-license.php) on https://code.google.com/p/dualsubsystem/ by [gaber...@gmail.com](https://code.google.com/u/112032256711997475328/).

This is not my software. It is provided here for reference, as Google Code will be closing and I have used some of this code for my project [ANSI|pipe](https://github.com/robhagemans/ansipipe). I do not take credit for dualsubsystem and neither do I support it.

Please note that the repository itself contains no files, as the original SVN repo was empty. Full source code is included with the binary in the [zip-file release](https://github.com/robhagemans/dualsubsystem/releases).

-------

## Summary ##
The windows kernel allows a program to be linked with `/SUBSYTEM:CONSOLE` or `/SUBSYSTEM:WINDOWS`.

Sometimes you would like a GUI application that also allows for command line interaction. This "opportunistic" console program is not natively supported. In a `CONSOLE` linked program, your GUI window will always be dependent on a console. If you launch from a shortcut, a new console is auto-created.

In a `WINDOWS` linked application, if you run it from a console, you have no access to the launching console's stdin/stdout/stderr pipes.

**dualsubsystem** is an update on a code written by Richard Eperjesi that provides the best of both worlds with the trick of using a same-named ".com" executable to handle the stdin/stdout/stderr redirection to the active console.

## References ##

First, the code used is adapted from codeguru article [Using the Console Like MSDEV](http://www.codeguru.com/cpp/w-d/console/redirection/article.php/c3955). I've been using this code for around 4 years in a production environment and have fixed some issues and updated it to compile on Visual Studio 2008.

One important fix was to allow for the ".com" launcher to be called with an absolute paths that contains spaces without mucking up the arguments it passes on to the `.exe`.

An MSDN blog also talks about this idea in an article by Junfeng Zhang called [How to make an application as both GUI and Console application?](http://blogs.msdn.com/junfeng/archive/2004/02/06/68531.aspx) referenced by another MSDN blog [How do I write a program that can be run either as a console or a GUI application?](http://blogs.msdn.com/oldnewthing/archive/2009/01/01/9259142.aspx).

Finally, there is a [stack overflow question](http://stackoverflow.com/questions/772089/one-executable-that-starts-as-gui-application-or-console-application-based-on-com) that is asking about this exact scenario with the question:

"Is there a way to set the compilation/linking such that the same executable can either present the gui windows or behave as console application based on command line parameters?"

## How It Works ##

Download the project and unzip. In the directory `example` is a precompiled `tester.com` and `tester.exe`. You can rename and use `tester.com` as-is in your project. Name it the same as your `.exe` and place it in the same directory when you deploy.

Using our example, when running `tester` in a command line, the command shell will use window's [order of precedence in locating executable files](http://support.microsoft.com/kb/35284) and run `tester.com` instead of `tester.exe`. `tester.com` sets up some named pipes and starts off a thread that will read/write from those pipe through to stdin/stdout/stderr. The `.com` process then launches the similarly named `.exe`, in this case `tester.exe`.

The just launched '.exe' must then have some initialization code to hook up stdin/stdout/stdin to the named pipes set up by the `.com`. It then will act like a `SUBSYSTEM:CONSOLE` application with respect to the current console.

## Example ##

The downloadable bundle contains precompiled binaries in the `example` folder.

The tester program requires some console input before launching a GUI application. If you run it from a console, it uses the current session. If you run it by launching `teser.exe` it creates a console to interact with.

Here is the output of running the `tester` example with some args.
```
C:\Development\DualMode\example>tester ReadMe.txt 20 --file="test file"
How old are you?
100
100? You still have a couple of years.
Command: C:\Development\DualMode\example\tester.exe ReadMe.txt 20 --file="test file"
```

The last line text after the `Command: ` prefix is the string returned by the Win32 `GetCommandLine()` function. You can see that all the command line arguments from the initial execution are retained in the call to the `.exe`.

## How to Use ##

Copy `tester.com` to your deploy directory and rename it to have the same base-name as your binary.

Copy only `DualModeI.h` and `DualModeI.cpp` from the `tester` directory into your project (feel free to rename). From your application, call the `InitializeDualMode()` method the `DualModeI.{h,cpp}` files provide. After this point, all stdin/stdout/stderr will be sent to the current active console (if there is one).

Your application can decide if it wants to create a attached console like `tester.exe` does when launched from a GUI context (i.e. not `cmd.exe`). To do this, pass `TRUE` as a parameter to `InitializeDualMode`. The default is `FALSE`. `InitializeDualMode()` will return `FALSE` if it was not able to connect to the named pipes. This return result can be ignored most of the time since you may expect your application to be launched from a GUI context and as such the `.com` would be running with the named pipes.
