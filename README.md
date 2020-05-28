# sdcsetup

BBC Micro GoSDC *SDCTool to run blocks of *commands

This tool is handy for making changes to your SDCCONFIG settings, as well running miscellaneous other *commands: by storing the commands in a *SDCTool means they're always available, rather than needing to "insert" a disc with a *EXEC file on, for example (this is the whole point of *SDCTools).

The two main use cases I have for it are:

* I have a RetroClinic MOS selector for my BBC Master - I want to easily select between appropriate configurations for the B, B+, Master 3.20 and Master 3.50 MOSs (with the associated SWROMs in those options)
* Switching between the normal (real) Acorn DFS+ADFS in the ROM and the GoSDC DFS+ADFS

I call the different configurations "setups" - rather than "configurations" - to avoid confusion with SDCConfig options.

## Changing setups

The code is designed to make it fairly easy to change the setups to ones that are useful for you.  The first thing to do is change the code and reassemble, then you can save and install.

### Change code and reassemble

Look for the line called _.setuptable_ - this is the list of setup names and addresses of the block of commands for that setup.  Each setup consists of:

> EQUS "M-SDC"
>
> EQUB -1
>
> EQUW setup_mastersdc

The _M_SDC_ is the name of the setup; the -1 indicates the end of the name and the _setup_mastersdc_ points to the address of the commands (see next).  The table is ended with:

> EQUB 0

The commands for the setup are defined below, for example:

> .setup_mastersdc
>
> \ MOS 3.50 with GoSDC DFS+ADFS
>
> EQUS "FSNR 6"+end_sdc$
>
> EQUS "F2NR 7"+end_sdc$
>
> EQUS "ROM4 2"+end_sdc$
>
> EQUS "UNPLUG 9"+end_os$
>
> EQUS "UNPLUG &D"+end_os$
>
> EQUS "CONFIGURE FILE 5"+end_os$
>
> EQUB 0

The _.setup_mastersdc_ is the same name used in the EQUW, above; the following line is just a comment (leading backslash).

The actual commands are listed following EQUS and terminated with either _+end_sdc$_ (which indicates the command should be preceded with "*SDCCONFIG" - so the first command above is really "*SDCCONFIG FSNR 6"), or _+end_os$_ (which indicates a complete, other *command).

The list of commands is finished with the "EQUB 0".

You assemble the new code with _RUN_.

### Test the setup

If you want to, you can test the setup, without having to install it first.

To do this, change the line near the top that reads:

> target%=&2000

into:

> target%=code%

Now RUN the program to assemble the code; you can test it by entering:

> PROCtest

This will display a prompt where you can enter parameters to the setup tool (i.e. the name of a setup, or nothing, to list the available setups).

Do NOT install this version of the code with *SDCTool as it will crash - change the _target%=_ line back to how it was and RUN again, first.

### Save and install

Save the machine code - the '_start+size load execute_' are displayed after assembly, right after the ampersand (&):

> *SAVE _filename start+size load execute_

If you have a previous version of the tool, remove it first with:

> *SDCTOOL SDCDEL T SETUP

Finally, install the new version of the tool:

> *SDCTOOL SDCATO _filename_ SETUP

You can then run the tool with:

> *SDCTOOL SETUP setup

This will run the commands for 'setup' - displaying them first and then executing them.  If you omit the final parameter, the available setups will be listed.
