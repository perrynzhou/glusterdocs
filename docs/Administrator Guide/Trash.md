Trash Translator
================
Trash translator will allow users to access deleted or truncated files. Every brick will maintain a hidden .trashcan directory , which will be used to store the files deleted or truncated from the respective brick. The aggregate of all those .trashcan directory can be accessed from the mount point. In order to avoid name collisions, a time stamp is appended to the original file name while it is being moved to trash directory.

## Implications and Usage
Apart from the primary use-case of accessing files deleted or truncated by user , the trash translator can be helpful for internal operations such as self-heal and rebalance. During self-heal and rebalance it is possible to lose crucial data. In those circumstances the trash translator can assist in recovery of the lost data. The trash translator is designed to intercept unlink, truncate and ftruncate fops, store a copy of the current file in the trash directory, and then perform the fop on the original file. For the internal operations , the files are stored under 'internal_op' folder inside trash directory.

## Volume Options

* ***`gluster volume set <VOLNAME> features.trash <on/off>`***

    This command can be used to enable trash translator in a volume. If set to on, trash directory will be created in every brick inside the volume during volume start command. By default translator is loaded during volume start but remains non-functional. Disabling trash with the help of this option will not remove the trash directory or even its contents from the volume.

* ***`gluster volume set <VOLNAME> features.trash-dir <name>`***

    This command is used to reconfigure the trash directory to a user specified name. The argument is a valid directory name. Directory will be created inside every brick under this name. If not specified by the user, the trash translator will create the trash directory with the default name “.trashcan”. This can be used only when trash-translator is on.

* ***`gluster volume set <VOLNAME> features.trash-max-filesize <size>`***

    This command can be used to filter files entering trash directory based on their size. Files above trash_max_filesize are deleted/truncated directly.Value for size may be followed by multiplicative suffixes as KB(=1024 bytes), MB(=1024\*1024 bytes) and GB(=1024\*1024\*1024 bytes). Default size is set to 5MB. Considering the fact that trash directory is consuming the glusterfs volume space, trash feature is implemented to function in such a way that it directly deletes/truncates files with size > 1GB even if this option is set to some value greater than 1GB.

* ***`gluster volume set <VOLNAME> features.trash-eliminate-path <path1> [ , <path2> , . . . ]`***

    This command can be used to set the eliminate pattern for the trash translator. Files residing under this pattern will not be moved to trash directory during deletion/truncation. Path must be a valid one present in volume.

* ***`gluster volume set <VOLNAME> features.trash-internal-op <on/off>`***

    This command can be used to enable trash for internal operations like self-heal and re-balance. By default set to off.

## Sample usage
Following steps give illustrates a simple scenario of deletion of file from directory

1. Create a simple distribute volume and start it.

        # gluster volume create test rhs:/home/brick
        # gluster volume start test

2. Enable trash translator

        # gluster volume set test features.trash on

3. Mount glusterfs volume via native client as follows.

        # mount -t glusterfs  rhs:test /mnt

4. Create a directory and file in the mount.

        # mkdir mnt/dir
        # echo abc > mnt/dir/file

5. Delete the file from the mount.

        # rm mnt/dir/file -rf

6. Checkout inside the trash directory.

        # ls mnt/.trashcan

We can find the deleted file inside the trash directory with timestamp appending on its filename.

For example,

```console
# mount -t glusterfs rh-host:/test /mnt/test
# mkdir /mnt/test/abc
# touch /mnt/test/abc/file
# rm /mnt/test/abc/file
remove regular empty file ‘/mnt/test/abc/file’? y
# ls /mnt/test/abc
#
# ls /mnt/test/.trashcan/abc/
file2014-08-21_123400
```

#### Points to be remembered
* As soon as the volume is started, trash directory will be created inside the volume and will be visible through mount. Disabling trash will not have any impact on its visibilty from the mount.
* Even though deletion of trash-directory is not permitted, currently residing trash contents will be removed on issuing delete on it and only an empty trash-directory exists.

#### Known issue
Since trash translator resides on the server side higher translator like AFR, DHT are unaware of rename and truncate operations being done by this translator which eventually moves the files to trash directory. Unless and until a complete-path-based lookup comes on trashed files, those may not be visible from the mount.
