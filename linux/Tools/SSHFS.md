# SSHFS

SSHFS allows you to mount a remote filesystem using SFTP. Most SSH servers support and enable this SFTP access by default, 
so SSHFS is very simple to use - there's nothing to do on the server-side.

Once sshfs is installed (see next section) running it is very simple:
```bash
shell> sshfs [user@]hostname:[directory] mountpoint
```

To unmount the filesystem:
```bash
shell> fusermount -u mountpoint
```

On BSD and macOS, to unmount the filesystem:
```bash
shell> umount mountpoint
```

* [Github官网地址](https://github.com/libfuse/sshfs)