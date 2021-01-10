# eudo

Executes a specified command or executable process with elevated administration rights, as according to the UAC security model
introduced by **Windows Vista**. The user will be prompted for elevated permissions to the specified process/command if
the user has UAC enabled.

### Usage

```
  Usage: eudo [switches] [--] [program] [args]
   -? | --help      Shows this help
   --wait           Waits until program terminates (default)
   --nowait         Runs the program and then immediately returns
   --hide           Hides the program from view; may not be honored by all programs
                    Hide is ignored when -k is specified.
   --show           Shows program/console window (default)
   -k               Invokes the specified command using %COMSPEC% /K
                    An interactive CMD prompt will remain open.
   -c               Invokes the specified command using %COMSPEC% /C
                    This does not offer any specific advantages over normal elevation and
                    is provided primarily for diagnostic purposes
   --version        Print app version to STDOUT and exit immediately.
   --verbose        Enables diagnostic log output.
  
   program          The program to execute; required unless -c|-k is specified
   args             command line arguments passed through to the program (optional)
```

Use `--` to stop options parsing and begin program-with-arguments parsing. This should be used when the target
executable filename begins with a dash or double dash, or if the executable is provided via variable.  Examples:

    $ eudo -- "-badfile.sh"
    $ eudo -- %~dp1

Use -k to open interactive command prompts such as a Visual Studio Tools Prompt.

    $ eudo -k "C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Auxiliary\Build\vcvars64.bat"

### Limitations of Windows Elevated Processes

Windows security model has some draconian restrictions about standard pipes and elevated processes. It does not allow
the standard output of an elevated process to be attached to a non-elevated terminal (console). Because of this, in
the common use-case scenario of a regular terminal, anything spawned by `eudo` will create its own console window,
and pipe redirections to file will not work.

#### Development remarks...

I am sad to confess this fact makes `eudo` largely irrelevant.

I'm thinking that `runas` verb of `ShellExecute` is not a good approach for this reason. If eudo itself had a manifest
trait requiring Administrator then Windows would prompt when trying to run eudo, and then eudo itself would have all
the permissions necessary to spawn elevated processes and tie their output back to the non-elevated console (in theory).

### Download / Install

*[todo!]*

### Why not call it `sudo`?

Elevate does not provide the ability to run processes as different users, therefore it is not analogous to `sudo` on Linux
systems. I believe that feature could be added to the utility in the future, however there are some caveats to that process
since technically Linux and Windows fundamentally differ in their security models -- Linux does not have a concept of *elevation*
of the current user. If that caveat can be resolved cleanly to allow for interoperable behavior between `bash/sh` scripts run on
either Windows/Linux OS, then at that point this utility could be re-released under the `sudo` moniker. Ideally such a utility
could then also be bundled with the likes of **MSYS/MinGW** or **Git for Windows**, though for that to happen the tool should be
adapted to use the MSYS CoreUtils framework, which has a ton of CLI advantages over Microsoft's CRT.

### History

This project was inspired by the somewhat-widely used `Elevate.exe` utility by Johannes Passing (hosted on github at https://github.com/jpassing/elevate).
I had wanted to use it for a build script that I was working on and discovered that it had a limitation that prevented it from
invoking or being invoked by from GIT-BASH shell scripts (see Git for Windows).

In the process of adding this feature, I realized that large swaths of the program could be vastly improved. I set about to
write my own, from scratch, with the following new features:

* file/path names and arguments lists are no longer limited to 260/520 characters
* handles argument quotations correctly
* easy way to specify path names that began with a forward slash (`/`) or dash (`-`)
* ability to execute via `COMSPEC` and exit (analogous to `cmd /c` )
* support for unix-style cmdline switchesm eg `--help`
* performs full windows filename extension association resolution -- allowing one to open `.txt` files via the registered text
  editor, or `.sh` files via **Git Bash**.
* Allows handling return codes from elevated processes


