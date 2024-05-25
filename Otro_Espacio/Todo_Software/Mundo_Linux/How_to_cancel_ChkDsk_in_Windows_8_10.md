# How to cancel ChkDsk in Windows 8/10
*by AnandK,* [*thewindowsclub.com*](http://thewindowsclub.com) *August 7, 2010*       


In this post we will see how to **cancel ChkDsk in Windows 8/10** at startup or reboot, after it has been scheduled in Windows.

[Disk Error Checking in Windows 8](http://www.thewindowsclub.com/disk-error-checking-windows-8)[Automatic Maintenance](http://www.thewindowsclub.com/automatic-maintenance-windows-8) and you now no longer need to really go and run it.

But there may be times when you may want to run the [command line version of ChkDsk](http://www.thewindowsclub.com/command-line-check-disk-windows-7).  In this case you may bring up CMD, enter the command **chkdsk /f /r c:** to identify bad sectors, attempt recovery of information and fix errors detected, and schedule a ChkDsk run at boot time.

**Cancel ChkDsk in Windows**

During Windows startup, you will be given a couple of seconds, during which you can **press any key** to abort the scheduled Disk checking.

If while it is running, you need to cancel it, then the only want to do it, is to **power down** the computer. But then, again on next restart, the scheduled chkdsk utility will run.

If you want to cancel chkdsk in Windows 8 after you have scheduled it, here is how you can do it.

**Using Registry Editor**

Run *regedit* to open the Registry Editor and navigate to the following key:

    HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager


Change the BootExecute entry from:

    autocheck autochk * /r\DosDevice\C:

To:

    autocheck autochk *

Close the Registry Editor and restart your WindowsPC. ChkDsk will not run.

**Using Command Prompt**

To cancel a scheduled check disk, open an elevated CMD windows, type the following and hit Enter:

    chkntfs /x c:

Here c is the drive letter. This should cancel the scheduled chkdsk run.

