Winject 1.7 alpha
(c) 2020 5R33CH4


Winject can inject a .dll or reset the debug port of a process.

Winject is designed to work on WinXP, Service Pack 2. The .NET Framework 2.0 has also resolved problems with resetting the debug port.

This is the recommended injector to use as it is not detected and does not hide from PunkBuster's detection in any way.



Winject is a simple DLL-injector/ejector with cmd-line support. 
It also includes built-in detector for OllyDbg and TSearch (more of this later).

_________________________________________________________________________________________________________
v1.7 New in this release (5/16/2020):

- You can now select target file via filedialog and inject via createprocess to _pre_ load DLL to target process
  this is needed for example D3D-hooks.



_________________________________________________________________________________________________________
v1.6 New in this release (6/8/2005):

- Winject is now able to reset _EPROCESS->DebugPort from other process. Just select process and go to [...] options
  for that process. If it's being debugged you can reset DebugPort to be able to attach new debuggr


_________________________________________________________________________________________________________
v1.5 New in this release (6.7.2005):

- Improved detector. Now also detects kernel-detours (you can still hide them if you are clever).
  Detector also checks for _EPROCESS->DebugPort and _EPROCESS->PEB->BeingDebugged flag for IsDebuggerPrecence()

- Improved minime.dll 






_________________________________________________________________________________________________________
v1.4 New in this release (1.7.2005):

- Stores used exe-name and DLL-name for next time defaults. Stored to registry ...CurrUser\Software\Winject. 
  If you clear tracking from File->Clear... the key AND ALL TRACES will be deleted and nothing is left behind.

- Autotracking for target. If target closes or is not open when Winject starts it will tracked and "locked" 
  into as soon as it's launched. NO autoinject - not now and never. Just use NUMPAD+ to inject when YOU wish. 

- Doubleway for injecting. First "clean" method and if it doesn't work try Detours. Still having problems 
  with some DLL's though.

- Some minor fixes and updates. 



_________________________________________________________________________________________________________
v1.3 New in this release (1.6.2005):
This is just a minor upgrade to support injecting and ejecting with hotkeys (NUMPAD +/-)

- /use cmdline-parameter. You can prefeed exename and dllname via shortcut or cmdline. 
  For example just make a shortcut for winject and add your gamename (/exename:bf1942.exe) 
  and hackdll (/dllname:c:\evil.dll) for arguments among with /use. 
  No need to choose the DLL & Process manually anymore from combo. Purpose is to support
  injecting with hotkeys when you have all hacks in a DLL and winject running as "trainer".

- using of Detours is removed. Injecting and Ejecting is now performed with custom-method (stub-function)

- hotkeys for inject (NUMPAD +) and eject (NUMPAD -). These are hardcoded - sorry. Call me lazy ;)

- When using hotkeys for injecting winject will automaticly re-acquire the PID for selected process. 
  Meaning that you only need to select process once (or use cmd-line prefeed). Winject will "lock" 
  to target-exe so that  it will automaticly re-acquire the ProcessID just before injecting. 
  The exe does NOT need to be running when launching winject - it can be launched later.

- Experimental mechanism for DLL's to request self-ejecing via winject. This allows your own DLL to 
  postmessage() to winject and request ejecting. To use this; first you need to disable ALL HW-breakpoints, 
  SEH, etc. etc. and then simply Postmessage() to winject (otherwise get ready for game-crash). 
  In the parameters you give your processID and modules (DLL) base-address.

  Just use following code to initiate self-ejecting. It's all you need:

  <CODE>
    MEMORY_BASIC_INFORMATION mbi;
    static int dummy;
    VirtualQuery( &dummy, &mbi, sizeof(mbi) );
    PostMessage(FindWindow(NULL," Winject"), WM_USER+1111, (WPARAM)GetCurrentProcessId(),(LPARAM)(DWORD)(mbi.AllocationBase));	
  <END CODE>

- bugfixes - none. Didn't receive any reports for bugs - nothing to fix :)


_________________________________________________________________________________________________________
v1.2 New in this release:
This is upgrade-release to support "Simple SEH II"-tutorial with hooks.
-inject via windowclass from cmd-line
-inject with SetWindowsHookEx()


Using is simple. Start winject, select process to inject into, select DLL you want to inject and click inject. The injection is verified automaticly by re-enumerating modules loaded in target process and checking the base-address.

if inject fails:
0 = LoadLibrary() Failed (bad behavior from DLL or protection in target)
1 = Process handle invalid
2 = DLLname validation failed
3 = VirtualAlloc for target process failed
4 = WriteProcessMemory (Dllname) failed
5 = CreateRemoteThread() failed
WAIT_TIMEOUT = Injected DLLMain() never returned until timeout (5 sec). 


CMD-line interface:
You can also use cmdline or shell from your own program to launch winject to inject DLL silently.

examples: 
winject /dllname:"d:\dlls\myhackdll.dll" /exename:bf1942.exe /silent
winject /dllname:"d:\dlls\hookhack.dll" /winclass:notepad /hook 
winject /dllname:"d:\dlls\stealdma.dll" /wintitle:notepad /eject


Valid parameters are:
 /dllname:"name and path of dll.dll"
 /exename:"executable name of target process.exe" or
 /wintitle:"WindowTitle of target process" or
 /winclass:"WindowClass of target process"or
 /pid:000 The ProcessID you get yourself

 /silent (skip all error-dialogs while injecting. only valid in cmdline-mode)
 /extened (use extended process access)
 /nosound (disable all annoying sounds)
 /eject (onloads spesified dll from target if possible)
 /hook (uses SetWindowHookEx() for injecting)
 /use (NEW! If you set /dllname and /exename -argumets and /use winject will preselect these automaticly for combos. 
       Note that the process does NOT need to be running. Winject will autofind the exe just before injecting)

Obviously you only use one of the ways to describe target-process (exename,wintitle,winclass or pid).


You can also create simple .cmd-file and shortcut to quickly perform the injecting:

[inject_to_game.cmd]
d:\tools\winject\winject.exe /dllname:d:\hacks\hookhack.dll /exename:mygame.exe /silent /hook



[HOOK]
New method for injecting is using SetWindowHookEx(). With this the OS will handle injecting automaticly.
The DLL needs to export InstallRemoteHook() -function which is called by Winject to initiate hooking.

__declspec( dllexport )  int InstallRemoteHook(DWORD dwThreadID);

The dwThreadID is passed from Winject for hook-DLL as target-thread to be hooked.
The hooking is done in 2 phases. First pseudo-hook is installed and from first callback the final
hook gets installed (and pseudo-hook is removed). This will free the DLL from launcher (winject) being running. 

Check tutorial "Simple SEH II" for example how to build DLL with hooks.


[PID-info]
You can enable extended process access from menu or via /extend -startup/cmd-line parameter.
This will simply AdjustTokenPrivileges to grant extra rights to winject so it can access to forbidden-processes
including critical system-processes. BE AWARE! You CAN even terminate client server subsystem (CSRSS) which will nicely simulate the effect of immediate master I/O-switch. 
From here you can also eject DLL's from selected process. The ejection is verified afterwards. If the module cannot be ejected it might still be in use.


[The Detector]
The thing with detector is to help to hide your hexedited debugger from target process (for examply OllyDbg or Tsearch). You need to hexedit the debugger to change window-title and class-names and exename. 
Then you can test if the debugger is really hidden by using winject built-in detector:
Open winject and attach your debugger to winject. Run test and see results. 
Next try to inject minime.dll into Winject itself and rerun tests while still having debugger attached.
The same effect will happen to actual target-process.


[minime.dll]
Winject currently includes only one dll to inject: minime.dll
You inject minime.dll INTO target process you wish to debug.
It only does 2 things:

 1) It patches Thread Information Block (TIB) which will "disable" IsDebuggerPresent() API from Kernel32.dll 
and also direct TIB lookup with inline asm (thanks test0r for cleaner solution :)

 2) It hijacks GetThreadContext() API from Kernel32.dll to always return fake values for debug-registers. 
You can use Hardware breakpoints with your debugger (again) and they are not detected.

TIP: You might want to rename minime.dll to some random name because after a while it is easy to detect by name from loaded modules.

[skype.dll]
disables skype from popping up while you are playing fullscreen game.
to remove simple eject or restart skype.


---------------------------------------------------------------------
Winject
Copyright (c) 2005 mcMike
This program is freeware

THE INFORMATION AND CODE PROVIDED IS PROVIDED AS IS WITHOUT WARRANTY
OF ANY KIND, EITHER EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO
THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR
PURPOSE. IN NO EVENT SHALL I BE LIABLE FOR ANY DAMAGES
WHATSOEVER INCLUDING DIRECT, INDIRECT, INCIDENTAL, CONSEQUENTIAL, LOSS
OF BUSINESS PROFITS OR SPECIAL DAMAGES, EVEN IF I HAS BEEN
ADVISED OF THE POSSIBILITY OF SUCH DAMAGES.
---------------------------------------------------------------------
Translated: Everything you do is upto you. I take NO responsibility whatsoever if shit happens.
