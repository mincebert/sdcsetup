# sdcsetup

BBC Micro GoSDC `*SDCTOOL` to run blocks of *commands

This tool is handy for making changes to your `*SDCCONFIG` settings, as well
running miscellaneous other *commands: by storing the commands in a `*SDCTOOL`
means they're always available, rather than needing to "insert" a disc with a
`*EXEC` file on, for example (this is the whole point of `*SDCTOOL`s).

The two main use cases I have for it are:

* I have a RetroClinic MultiOS for my BBC Master - I want to easily select between appropriate configurations for the B, B+, Master 3.20 and Master 3.50 MOSs (with the associated SWROMs in those options)
* Switching between the normal (real) Acorn DFS+ADFS in the ROM and the GoSDC DFS+ADFS

I call the different configurations "setups" - rather than "configurations" -
to avoid confusion with `*SDCCONFIG` options and as a good name for the tool.

## Files

Three main files are included in this distribution:

* `b.sdcset` - the tokenised BASIC program that assembles the tool: you can edit this to modify the available setups
* `t.sdcset` - a textual version of `b.sdcset` that can be (after using `lf2cr` - see below) `*EXEC`d to enter the program - it's useful primarily for examing the code and seeing differences in Git
* `x.sdcset` - the machine language code assembled by `b.sdcset`: it can be loaded directly into GoSDC - see the _Use_ section, below
* `mksdcs` - a BBC file that can be *EXECd to assemble the code in `b.sdcset` and output `t.sdcset` and `x.sdcset`

In addition, there are a couple of utilities in the `bin` directory that you can pipe files through:

* `spool2exec` - this takes a listing of the BASIC program `t.sdcset`, generated by `mksdcs` on the BBC and converts it to the version in this repository (stripping the line numbers and adding a header)
* `cr2lf` - this converts BBC text (&0D/CR line endings) to Unix text (&0A/LF line endings); used to convert `mksdcs` from the BBC
* `lf2cr` - this converts Unix text to BBC text; it is used to convert the `t.sdcset` and `mksdcs` text into a file that can be `*EXEC`d on the BBC

## Use

Note that the `*SDCTOOL` utility requires that any Tube processor (if present)
is disabled, so all the commands must be executed running only from the
internal processor.  Once the command has executed, however, the Tube processor
can be re-enabled.

Install the tool (needs only to be done once and is stored in the GoSDC flash
area):

    *SDCTOOL SDCATO X.SDCSET SETUP

You can list the available setups by running:

    *SDCTOOL SETUP

You can select a particular setup with (e.g.):

    *SDCTOOL SETUP M

You will need to know what the setup names mean - the BASIC code that assembles
the tool describes them in comments preceeded with blackslashes (see _Change code and reassemble_, below).  In this case `M` is my Master MOS 3.50 configuration.

## Modifying setups

The code is designed to make it fairly easy to change the setups to ones that
are useful for you.  The first thing to do is change the code and reassemble,
then you can save and install.

### Change code and reassemble

Look for the line called `.setuptable` - this is the list of setup names and
addresses of the block of commands for that setup.  Each setup consists of:

    EQUS "M-SDC"
    EQUB -1
    EQUW setup_mastersdc

The `"M-SDC"` is the name of the setup; the `-1` indicates the end of the name
and the `setup_mastersdc` points to the address of the commands (see next).

The table is ended with:

    EQUB 0

The commands for the setup are defined below, for example:

    .setup_mastersdc
    \ MOS 3.50 with GoSDC DFS+ADFS
    EQUS "FSNR 6"+end_sdc$
    EQUS "F2NR 7"+end_sdc$
    EQUS "ROM4 2"+end_sdc$
    EQUS "UNPLUG 9"+end_os$
    EQUS "UNPLUG &D"+end_os$
    EQUS "CONFIGURE FILE 5"+end_os$
    EQUB 0

The `.setup_mastersdc` is the same name used in the EQUW, above; the following
line (with the leading backslash) is just a comment explaining its purpose.

The actual commands are listed in the string following EQUS and terminated
with either:

- `+end_sdc$` (which indicates the command should be preceded with `*SDCCONFIG `
(including trailing space) so the first command above is really `*SDCCONFIG
FSNR 6`), or
- `+end_os$` (which indicates a complete, other *command).

The list of commands is finished with the `EQUB 0`.

You assemble the new code with `RUN`.

### Test the setup

If you want to, you can test the setup, without having to install it first.

To do this, change the line near the top that reads:

    test%=FALSE

into:

    test%=TRUE

Now `RUN` the program to assemble the code - it will be assembled at a
location in memory it can be run from directly (not &2000, where `*SDCTOOL`
would load it) and a test routine will be automatically entered, after
assembly.

This will display a prompt where you can enter parameters to the setup tool
(i.e. the name of a setup, or nothing, to list the available setups).

Do NOT install this version of the code with `*SDCTOOL` as it will crash -
change the line back to `test%=FALSE` and `RUN` again, first.

### Save and install

Save the machine code - an example command line with *SAVE is shown after
running the program in non-test mode.

If you have a previous version of the tool installed, remove it first with:

    *SDCTOOL SDCDEL T SETUP

Finally, install the new version of the tool, e.g.:

    *SDCTOOL SDCATO X.SDCSET SETUP

You can then run the tool with:

    *SDCTOOL SETUP setup

This will run the commands for _setup_ - displaying them first and then
executing them.  If you omit the final parameter, the available setups will be
listed.
