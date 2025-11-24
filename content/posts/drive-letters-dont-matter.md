+++
title = "Drive Letters don't matter"
date = "2025-11-24T01:03:00Z"
description = "Windows drive letters don't matter, and not assigning one won't stop reads and write to filesystems"
+++

**tldr:** set unwanted disks to offline. if you want to protect another partition on your OS drive, too bad.

So I've just learnt something fun which makes me question the separation of Windows installs on my PC. Windows doesn't need drive letters to access filesystems. I assumed if you don't set a mount point (drive letter or folder path) then a filesystem was inaccessible, but it turns out that's not the case.

```powershell
Get-Volume | Select-Object DriveLetter, FileSystemLabel, Path

DriveLetter FileSystemLabel Path
----------- --------------- ----
            Test            \\?\Volume{82df1155-2db0-4bd2-a437-4a91cbb05d10}\
C           Windows         \\?\Volume{de3be8f7-1617-4ca4-a360-4c10abfef865}\
```

See the above PowerShell; I have two volumes on my system. One that's `Windows` on `C:`, and another labelled `Test` that's not mounted. But what's the `Path` property?

Well the `Path` is (unsurprisingly) the path to the volume. Replace the `?` with `.` and you'll be able to read and write to the unmounted filesystem.

```powershell
PS C:\Users\Dan> "Hello World" > "\\.\Volume{82df1155-2db0-4bd2-a437-4a91cbb05d10}\test.txt"
PS C:\Users\Dan> dir "\\.\Volume{82df1155-2db0-4bd2-a437-4a91cbb05d10}\"


    Directory: \\.\Volume{82df1155-2db0-4bd2-a437-4a91cbb05d10}


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        24/11/2025     00:48             28 test.txt



```

If you really want a volume to be untouchable, you've got to set it to `offline` in Disk Management or with the `Set-Disk -IsOffline $true` cmdlet. I guess not setting them to `offline` could lead to malware spreading from one OS install to another.