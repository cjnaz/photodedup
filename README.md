# photodedup - Purge duplicate photo and video files

This tool is used for purging (deleting) duplicate files from your directory tree(s).

## Context and primary usage
- I take photos and videos mostly on my OnePlus 6T phone.  They land in `/DCIM/Camera`.  I create folders within `/DCIM` for specific activities (such as `/DCIM/Dunes`) in an effort to keep the Camera folder sane.
- I periodically use the wonderful [FolderSync Pro](https://play.google.com/store/apps/details?id=dk.tacit.android.foldersync.full) on my phone to push the entire `/DCIM` tree up to the file server on my LAN.  I set up FolderSync to push the entire `/DCIM` tree "To remote folder" so that no modifications are ever done on the phone itself, and the DCIM tree on the LAN only _accumulates_ files (never deleted by FolderSync).  Note that files will accumulate `DCIM/Camera` folder on the LAN.
- On the phone I use the wonderful [Simple Gallery Pro - Photo Manager & Editor](https://play.google.com/store/apps/details?id=com.simplemobiletools.gallery.pro) as my gallery app.  Periodically, when I feel like organizing photos/videos on my phone I'll look through the Camera folder and move photos/videos to existing or new folders within `/DCIM`, such as `/DCIM/Dunes`.  
- _**Now we have a problem...**_  _Since FolderSync runs push the full `/DCIM` tree to the LAN file server I'll tend to accumulate copies of photos/videos in both the Camera and the Dunes folders._  Without some scheme to control the mess, over time I'll end up with tons of duplicate files.
- **photodedup is used to find all duplicate files within the LAN `DCIM/` tree and optionally delete duplicates that exist in the `DCIM/Camera` folder.**

## Usage notes and features
- The primary usage is to delete files in the LAN-side `DCIM/Camera` folder that also exist in other folders within `DCIM/`, as described above.
- Nothing is deleted unless the `--purge` switch is specified.  Without `--purge` duplicates are only listed.
- You may have other photo trees from prior phones and backups on your hard disk.  `--second-tree` mode supports deleting duplicate files from these older/other backups that also exist in the _main_ tree (`main_root`).  In the `Two tree mode, and --no-time and --no-size switches` example below the file `China Wall Video.MOV` would be deleted from the secondary tree folder `/mnt/share/media/Pictures/Dunes/Dan`, and retained in the main tree folder `/mnt/share/media/Chris phone pix/Dunes`.
- photodedup works on all file types, not ust media files.  When using the `--second-tree` mode duplicates found in both the main tree and the second tree may be deleted from the second tree regardless of file type/extension.  Note that single tree mode specifically looks for duplicates relative to the `DCIM/Camera` folder, so this mode is pretty photo/video specific.  Other tools for finding duplicates may be better suited to your needs, including CCleaner and Beyond Compare.
- Some evil file management tools clobber the file modification datetime on files.  (Both FolderSync and Simple Gallery are well behaved.)  This gives rise to duplicate photos that in the trees that have different modification times.  The `--no-time` switch may be specified to disable matching on datetime.  See `Two tree mode, and --no-time and --no-size switches` example below for file `IMG_20140426_115922_309.jpg`.
- Eventually you'll have files from different cameras that have the same filename but different datetime and size.  photodedup will recognize these as different files and not attempt any purges.  Alternately, you may edit a file and save it to the same filename, producing a duplicate with a different size.  To identify the original and edited versions as duplicates specify the `--no-size` switch along with the `--no-time` switch.  See `Two tree mode, and --no-time and --no-size switches` example below for file `DSCF2363.JPG`.  Three different versions of this file exist, some with differing datetime stamps.  Manually deleting these duplicates may be most appropriate.
- Various tools may create `.Thumbnails` subdirectories.  These may be deleted using the `thumbs` switch.
- photodedup was developed and tested on Linux and Python 3.  It is _**not**_ supported on Python 2.7, but seems to run fine.  Issues may be encountered with file/path names with Unicode characters / code points.  If you have issues with Python 2.7, please use Python 3.


## Changes in the most recent release
- 200306 v0.1  New

## Setup
- Simply place the script file photodedup in a folder in your path environment variable, or run it while specifying the full path to the file.  

## CLI

```
$ ./photodedup -h
usage: photodedup [-h] [--no-size] [--no-time] [--second-tree SECOND_TREE]
                  [--thumbs] [--purge] [-V]
                  main_root

Find and delete duplicate files within a specified tree.

positional arguments:
  main_root             Root of the main tree.  In single tree mode, duplicates in <main_root>/Camera will be deleted.

optional arguments:
  -h, --help            show this help message and exit
  --no-size             Disregard file size when identifying duplicates.  Size is included by default.
  --no-time             Disregard file modification time when identifying duplicates.  Time is included by default.
  --second-tree SECOND_TREE
                        Root of other archive tree.  Duplicate files will be deleted from this tree.
  --thumbs              Find .Thumbnails directories.  Delete with --purge.
  --purge               Print status, but copy/delete no files.
  -V, --version         Return version number and exit.

```

## Example runs
### Single tree mode
```
$ ./photodedup /mnt/share/media/Chris\ phone\ pix/
Checking for files with MATCHING timestamp and MATCHING size.
File <20170321_221733-1.jpg> has 2 copies
    Tue Mar 21 23:18:44 2017     2032139 bytes    /mnt/share/media/Chris phone pix/Pets
    Tue Mar 21 23:18:44 2017     2032139 bytes    /mnt/share/media/Chris phone pix/Camera
File <20181022_114054.jpg> has 2 copies
    Mon Oct 22 11:40:54 2018     7732960 bytes    /mnt/share/media/Chris phone pix/Pets
    Mon Oct 22 11:40:54 2018     7732960 bytes    /mnt/share/media/Chris phone pix/Camera
File <20181102_160747.mp4> has 2 copies
    Fri Nov  2 16:08:24 2018    69911939 bytes    /mnt/share/media/Chris phone pix/Camera
    Fri Nov  2 16:08:24 2018    69911939 bytes    /mnt/share/media/Chris phone pix/Dunes
...
Found  25  duplicate files.  Deleted  0  duplicate files.
```

### Two tree mode, and `--no-time` and `--no-size` switches
```
$ ./photodedup /mnt/share/media/Chris\ phone\ pix/ --second-tree /mnt/share/media/Pictures --no-time --no-size
Checking for files with ANY timestamp and ANY size.
Operating on --second-tree only
Found instance(s) of File <China Wall Video.MOV> in second tree that exist in the main tree:
    Main tree:    Sat Feb  7 12:09:20 2015   100288778 bytes    /mnt/share/media/Chris phone pix/Dunes
    Second tree:  Sat Feb  7 12:09:20 2015   100288778 bytes    /mnt/share/media/Pictures/Dunes/Dan
...
Found instance(s) of File <IMG_20140426_115922_309.jpg> in second tree that exist in the main tree:
    Main tree:    Sat Apr 26 11:59:22 2014     1526723 bytes    /mnt/share/media/Chris phone pix/HPVC'14
    Second tree:  Mon Apr 28 10:26:31 2014     1526723 bytes    /mnt/share/media/Pictures/NAU HPVC 2014
...
Found instance(s) of File <P9110100.JPG> in second tree that exist in the main tree:
    Main tree:    Tue Sep 11 11:04:06 2018     2988468 bytes    /mnt/share/media/Chris phone pix/CO Springs
    Second tree:  Tue Sep 11 11:04:06 2018     2988468 bytes    /mnt/share/media/Pictures/CO Springs Sep'18
...
Found instance(s) of File <DSCF2363.JPG> in second tree that exist in the main tree:
    Main tree:    Sat Dec 31 11:48:24 2016     4146999 bytes    /mnt/share/media/Chris phone pix/Bottlebrush
    Second tree:  Sun Oct  4 17:53:38 2009      809034 bytes    /mnt/share/media/Pictures/not
    Second tree:  Sun Aug  5 20:28:50 2007      803603 bytes    /mnt/share/media/Pictures/Mabul07
    Second tree:  Sun Aug  5 20:28:50 2007      803603 bytes    /mnt/share/media/Pictures/Mabul07/Select
    Second tree:  Wed Apr 10 13:50:56 2013     4146999 bytes    /mnt/share/media/Pictures/Bottlebrush
    Second tree:  Mon Dec 21 14:45:56 2009     3119118 bytes    /mnt/share/media/Pictures/Family gatherings/Xmas 09

```

## Known issues
- none

## Revision history
- 200306 v0.1  New
