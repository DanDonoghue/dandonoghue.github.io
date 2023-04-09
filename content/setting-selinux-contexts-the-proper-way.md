+++
title = "Setting SELinux Contexts the Proper Way"
date = "2016-05-02T05:37:00Z"
description = "SELinux is confusing, especially when almost every example is bullshit. This is a shitty attempt at saying the proper way to set SELinux contexts on files and directories."
+++

Almost every mention of SELinux Contexts online references the use of `chcon(1)` for setting contexts on file and directories. This is a pretty shit way of setting contexts, since these changes don't stay when the file system is relabeled, and with `chcon(1)` the contexts need to be manually set again after an OS reinstallation.

The proper way to set contexts is to use `semanage(8)`, which appends a new line with the rule to `/etc/selinux/targeted/contexts/files/file_contexts.local`. Real useful, since the relabelling of a file-system will follow whatever rules are specified within this file, and it's far easier to copy the `file_contexts.local` file than it would be to manually run `chcon(1)` after a relabel/reinstallation.

Setting contexts with `semanage(8)` and `restorecon(8)` is actually a lot less effort than using `chcon(1)`, mainly because you don't have to specify the context after the first command. I'll give an example. Say you have a directory `/mnt/storage/My Files` that you want to share via Samba, well it needs the `samba_share_t` context. To give it this context with `chcon(1)` you'd run `chcon -t samba_share_t /mnt/storage/My\ Files`, which does work. Now you're copying a load of files from some other directory into the "My Files" directory, these all need the `samba_share_t` context, so you've got to run `chcon -t samba_share_t /mnt/storage/My\ Files -R` to set the correct context. That's a bit of a mouthful.

Using `semanage(8)`, this could be reduced down to running `semanage fcontext -a -t CONTEXT "REGEX"` once, and then running `restorecon -R PATH`. That's a lot less typing for the same effect, with the benefit of the rule being permanent, and transferable between reinstallations. The use of regular expressions is the only real part that's slightly more complex than using `chcon(1)`, but if you're playing about with SELinux then you should be the kind of user who knows or is learning regex anyway. Most of the paths you'd want to use are either using wildcards `(.*)`, or spaces `[\s]`. The example below uses `(\.*)`, which matches both the parent directory and its contents, basically a short way of saying both `/mnt/storage/My Files` and everything inside of it.

```
semanage fcontext -a -t samba_share_t "/mnt/storage/My[\s]Files(\.*)?"
restorecon -R /mnt/storage/My\ Files
```

Another useful rule is for the custom home-dir location. Say home directories are actually stored at `/mnt/storage/home`, you can run the following to get the context of this directory to persist.

```
semanage fcontext -a -t home_root_t "/mnt/storage/home"
semanage fcontext -a -t user_home_t "/mnt/storage/home/(.*)?"
```