# ChkDsk stuck at particular % or hangs at some Stage
If you find that Check Disk or ChkDsk is stuck at a particular percentage or hangs at some stage in Windows, then this post suggests some troubleshooting steps that may help you. It could be 10%, 12%, 27% or any such percentage. Again, it could be stage 2, 4, 5 or any such.

**ChkDsk stuck or hangs**

1] The best suggestion I have to give is hang on and let it run. It may take a couple of hours, but given time, it it is known to complete in most cases. If need be, leave it overnight and let it run its course.

2] If this does not help, restart your computer, by pressing the power button. During the next boot, press the Esc, Enter or the appropriate key to stop the running of ChkDsk. Once you boot to the desktop, do the following:


- Run [Disk Cleanup utility](http://www.thewindowsclub.com/disk-cleanup-utility-windows) to clear your junk files.
- Open an elevated CMD type **sfc /scannow** and hit Enter to run the [System File Checker](http://www.thewindowsclub.com/how-to-run-system-file-checker-analyze-its-logs-in-windows-7-vista). Once the scan is completed, restart your computer. Remember to exit ChkDsk during boot.
- Next, again open CMD as admin type **Dism /Online /Cleanup-Image /RestoreHealth** and hit Enter to [repair the Windows image](http://www.thewindowsclub.com/component-store-corruption-repair-windows-image).

Now see if ChkDsk is able to complete the scan. As I mentioned earlier, keep it over night if need be.

*Hope it helps.*

This problem occurs more in Windows 7 and earlier. Windows 8 and Windows 10 handles disk check operations more efficiently. [Disk Error Checking](http://www.thewindowsclub.com/disk-error-checking-windows-8)[Automatic Maintenance](http://www.thewindowsclub.com/automatic-maintenance-windows-8) and you now no longer need to really go and run it.

It is important that you keep a watch on your [Hard Disk health,](http://www.thewindowsclub.com/hard-disk-drive-health) and thus imperative that ChkDsk complete its run. But if you wish to, you can [cancel the ChkDsk operation](http://www.thewindowsclub.com/how-to-cancel-chkdsk-in-windows-8).

