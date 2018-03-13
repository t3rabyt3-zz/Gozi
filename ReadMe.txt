ISFB is a bot program designed to analyze and modify HTTP traffic on the client's computer.

Supports all 32 and 64 bit Windows, starting with Windows XP.
Supports all 32 and 64 bit versions of Internet Explorer, starting with 6.0.
Supports all 32 and 64 bit versions of Mozilla Firefox.
Supports all 32-bit versions of Google Chrome.

The program is able to install and work without the privileges of the administrator.
Handles all HTTP traffic of the browser including encrypted HTTPS.

The bot is managed from a remote server, using configuration files and commands.
Configuration files and commands are signed using RSA. When receiving files, the bot checks the digital signature,
 and, in case of non-compliance of the signature, the file is ignored.

At the first start, the bot initiates a timer. In the future, on a timer, the bot refers to the managing server for the files.

There are 2 ways to find the management server:
- search through the specified list of domain names and select the active one;
- generation of a dynamic list of domain names depending on the current date and system configuration;

Traffic analysis is performed on the basis of a specially generated configuration file, which the bot receives from the server.
Such a file can contain the following instructions:
- substitution of the entire HTML page
- replacement of a fragment of HTML page
- copy the fragment of the page and send it to the server
- find the file by mask and send it to the server
- make a screen shot and send to the server

In addition to the configuration file, the bot receives commands from the server:
GET_CERTS - export and send certificates installed in the Windows system store. 
			For XP unloads, as well, non-exportable certificates.
GET_COOKIES - collect cookies FF and IE, Flash SOL-files, package them with preservation of structure 
			directories and send to the server.
CLR_COOKIES - delete cookies FF and IE, SOL-files Flash.
GET_SYSINFO - collect system information: processor type, OS version, process list, list
			drivers, the list of installed programs.
KILL - kill the OS (only works with administrator rights)
REBOOT - reboot OS
GROUP = n - change bot group ID to n
LOAD_EXE = URL - download the file from the specified URL and launch it
LOAD_REG_EXE = URL- upload the file from the specified URL, register it in autirun and run
LOAD_UPDATE = URL - load the program update and run
GET_LOG - send the internal log to the server
GET_FILES = * - find all the files matching the specified mask and send to the server
SLEEP = n - stop processing the command queue for n milliseconds. (used for long operations)
SEND_ALL - Send all the data from the queue to send immediately. Otherwise, the data is recovered
			by the timer.
LOAD_DLL = URL [, URL] - download the specified DLL URL and inject it into the explorer.exe process.
			The first URL for a 32-bit DLL, the second for a 64-bit DLL.

SOCKS_START = IP: PORT - start the server4 (if available)
SOCKS_STOP - stop the x4 \ 5 server

GET_KEYLOG - send the data of the keylogger (if available)
GET_MAIL - activate grabber E-Mail (if available) and send received data from it
GET_FTP - activate FTP grabber (if available) and send data received from it

SELF_DELETE - remove software from the system, including all files and registry keys

URL_BLOCK = URL - block access to all URLs that match the specified mask
URL_UNBLOCK = URL - Unblock access to the URL that matches the specified mask, previously blocked by the URL_BLOCK command
FORMS_ON - enable the HTTP form grabber (if there is _ALWAYS_HTTPS define, the HTTPs grabber is always on)
FORMS_OFF - disable the HTTP form grabber
KEYLOG_ON [= list] - enable the keylog for the specified process list
KEYLOG_OFF - disable the keylogger
LOAD_INI = URL - load the packed INI-file from the specified URL, save it in the registry and use it instead of the INI-file,
					attached to the software with the help of the builder. The INI file must be packed and signed.
					
LOAD_REG_DLL = name, URL [, URL] - upload the DLL to the specified URL, save it under the specified name and register for
					automatic boot after each system startup
UNREG_DLL = name - remove from automatic load DLL with given name

Technical details

Dropper - installation program.
The dropper is a Windows executable (PE32). In the file, in the form of a binary resource, are contained
two packed DLL: 32-bit and 64-bit bot.
At startup, the dropper decompresses the DLL and registers them for autorun. 
DLLs are unpacked and registered in such a way that they can be executed at any level of privileges: 
 both with the administrator and with the user.

DLL - bot.
A bot is a dynamically loadable library (DLL). For each architecture, its own corresponding DLL is compiled.
The DLL bot is loaded into all the processes that are started.
The bot consists of 2 logical components: the parser and the server. The parser is activated in the context of the browser process.
The server is activated in the context of the shell process (usually explorer.exe).

The parser performs the following functions:
- sending / receiving data (receiving commands, configs, sending forms, files)
- direct interception, analysis, and modification of HTTP traffic

The server (in the context of explorer.exe) executes:
- file operations (search, create and delete files)
- launching programs, updating
- system functions (reboot, OS lock)

Thus, all operations that require privileges are performed by the server in the context of explorer.exe, 
and all operations with the network exclusively from the browser.


Build and configure

The project is built using Microsoft Visual Studio 2005, or a later version.
The project integrates the cryptor, which is used by default. 
As a result of the assembly and crypt, the following files are obtained:
Release \ crm_p.exe
Release \ client_p.dll
x64 \ Release \ client_p.dll
it's packaged and encrypted versions of the bot and dropper, and the dropper (the crm_p.exe file) contains the other two.
The uncritical versions of the bot are in the same place:
Release \ crm.exe
Release \ client.dll
x64 \ Release \ client.dll

In addition to the bot, the project includes:
Release \ dname.exe - utility for generating pseudo-random domain names;
Release \ rsakey.exe - utility for signing commands and config files;
config.exe is the program configurator.

The basic settings of the program are in the files id.h and config.h.
id.h contains the bot group number.
config.h contains such parameters as: list of control servers, names of URLs for receiving commands and configs,
 and for sending data, as well as various keys and parameters that affect the configuration of the program.
 

Build with builder

It is possible to compile the ISFB so that you can later attach the keys and configuration files to the DLL,
 not rebuilding the project.

1. Build ISFB in Release (Builder) configuration under x86 and x64.
2. Edit the files: \ public.key and \ client.ini, containing the RSA-key and the program settings, respectively.
3. In the console window, run build.bat from the \ Builder folder
4. Take the ready-made installer.exe from the \ Builder \ Release folder

The batch file build.bat starts the builder, which attaches to the DLL (for x86 and x64) files: 
 public.key and client.ini.
Later both DLLs are attached to the installer.
The ready installer is saved to the file \ Release \ install.exe


Assembling with BK

It is possible to assemble the ISFB together BK into one executable installer, so that in the case
 errors when installing BK, the installer removed the DLL and installed them separately.
Note: the folder containing the BK2 solution must be in the same directory as the folder containing the ISFB.

1. Collect BK in Release configuration under x86 and x64.
2. Build ISFB in Release (Builder) configuration under x86 and x64.
3. Edit the files: \ public.key and \ client.ini.
4. In the console window, run bkbuild.bat from the \ Builder folder
5. Pick up the assembled bksetup.exe that contains BK, ISFB-DLL and ISFB installer from \ Builder \ Release


Operation in injection mode from memory

To work in the injection mode from memory, set the value of the _INJECT_AS_IMAGE flag in the file \ common \ main.h in the
 TRUE, and rebuild the project. In this case, the installer does not create the DLL on the disk, but copies itself to one of the system folders
 and registered in Windows AutoRun. 
When you run the installer, you inject the DLL image, the corresponding architecture, into Explorer.exe, from which, in turn,
 Corresponding image DLL inektitsya in all affected processes, different architectures.


Plugins

ISFB supports plugins: specially assembled, DLL exporting PluginRegisterCallbacks function and calling
 internal software functions (for example, data sending functions).
To load the plugin, use the command:
 LOAD_PLUGIN = URL [, URL] - where the first URL for the 32-bit version of the DLL, the second - 64x-bit.
Soft downloads the DLL of the corresponding architecture and injects it into explorer.exe, then the function is called 
 PluginRegisterCallbacks, which is passed a pointer to the list of callbacks (functions) implemented
 inside the software that the plug-in can use.
Description of structures and prototypes of functions for creating plug-ins is found in the file \ common \ plugin.h


Structure of the project

\ AcDLL - library of injections. It implements the DLL injection mechanism in all affected processes, regardless of architecture.
	Supports two modes of operation: inject, directly DLL and inject DLL from memory without creating a file on disk.
\ ApDepack is an APLIB-based library that releases unpacking functions.
\ BcClient - client library for the back end of the server.
\ Client - main application DLL
\ Common - a bibliotech library that implements common functions used in different parts of the project. Such as: reading files, registry keys,
	operations with data streams, with strings, with XML, hooks, and so on.
\ Crypto - library of cryptographic functions. It implements the following algorithms: CRC32, BASE64, MD5, RSA, RC6, AES, DES, SHA1.
	Used to sign config files and command files, as well as to encrypt e-mail and ftp accounts.
\ Dname - the program for generating domain names based on the group number of the software and the current date.
\ Ftp - FTP-grabber library.
\ Handle - library that implements the hash table. Used to bind HTTP request handles to the internal ISFB context.
	Also, it is used by a keylogger to group keyboard logs by PIDs and HWNDs.
\ IM - DLL-plugin that implements Instant Messangers grabber.
\ Install - ISFB installer.
\ KeyLog - library keylogger.
\ Mail - the library of E-mail grabbers.
\ RsaKey is a program for encrypting and digitally signing config files and command files.
\ SocksLib - library that implements SOCKS4 \ 5-server.
\ Sqlite3 - library for working with the SQLLite database. Used by IM grabbers.
\ ZConv - Zeus configurations converter program into ISFB config files.