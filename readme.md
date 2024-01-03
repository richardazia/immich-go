# Bulk Uploading to Immich with `immich-go`

Do you have a large collection of photos or extensive Google Photos takeout files to upload in 'immich'?<br>
Are you struggling with Node.js or Docker installation just to upload your photos to 'immich'?

Give a try to the `immich-go` tool.

- import from folder(s).
- import from zipped archives without prior extraction.
- discard duplicate images, based on the file name, and the date of capture.
- import only missing files or better files (an delete the inferior copy from the server).
- import from Google Photos takeout archives:
    - use metadata to bypass file name discrepancies in the archive
    - use metadata to get album real names
    - use date of capture found in the json files
    - create albums based on Google Photos albums or folder names.
- import photos taken within a date range.
- import and stack couples jpg/raw photos or bursts
- import IPhone live photos
- remove duplicated assets, based on the file name, date of capture, and file size
- no installation, no dependencies.

> ⚠️ This an early version, not yet extensively tested<br>
> ⚠️ Keep a backup copy of your files for safety<br>


For insights into the reasoning behind this alternative to `immich-cli`, please read the motivation [here](docs/motivation.md).


## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=simulot/immich-go&type=Date)](https://star-history.com/#simulot/immich-go&Date)


# Executing `immich-go`
The `immich-go` program uses the Immich API. Hence it need the server address and a valid API key.


```sh
immich-go -server URL -key KEY -general_options COMMAND -command_options... {files}
```

`-server URL` URL of the Immich service, example http://<your-ip>:2283 or https://your-domain<br>
`-api URL` URL of the Immich api endpoint (http://container_ip:3301)<br>
`-skip-verify-ssl <bool>` Skip SSL verification for use with self-signed certificates (default: false)

`-key KEY` A key generated by the user. Uploaded photos will belong to the key's owner.<br>
`-no-colors-log` Remove color codes from logs.<br>

`-log-level` Adjust the log verbosity as follow: (Default OK) <br>
- `ERROR`: Display only errors
- `WARNING`: Same as previous one plus non blocking error
- `OK`: Same as previous plus actions
- `INFO`: Same as previous one plus progressions <br>

`-log-file=file` Write all messages to the file<br>
`-time-zone=time_zone_name` Set the time zone<br>

## Command `upload`

Use this command for uploading photos and videos from a local directory, a zipped folder or all zip files that google photo takeout procedure has generated.

### Switches and options:
`-album "ALBUM NAME"` Import assets into the Immich album `ALBUM NAME`.<br>
`-device-uuid VALUE` Force the device identification (default $HOSTNAME).<br>
`-dry-run` Preview all actions as they would be done.<br> 
`-create-album-folder <bool>` Generate immich albums after folder names (default FALSE).<br>
`-force-sidecar <bool>` Force sending a .xmp sidecar file beside images. With Google photos date and GPS coordinates are taken from metadata.json files. (default: FALSE).<br>
`-create-stacks <bool>`Stack jpg/raw or bursts (default TRUE).<br>
`-stack-jpg-raw <bool>`Control the stacking of jpg/raw photos (default TRUE).<br>
`-stack-burst <bool>`Control the stacking bursts (default TRUE).<br>
`-select-types .ext,.ext,.ext...` List of accepted extensions. <br>
`-exclude-types .ext,.ext,.ext...` List of excluded extensions. <br>

### Date selection:
Fine-tune import based on specific dates:<br>
`-date YYYY-MM-DD` import photos taken on a particular day.<br>
`-date YYYY-MM` select photos taken during a particular month.<br>
`-date YYYY` select photos taken during a particular year.<br>
`-date YYYY-MM-DD,YYYY-MM-DD` select photos taken within this date range.<br>

### Google photos options:

Specialized options for Google Photos management:<br>
`-google-photos` import from a Google Photos structured archive, recreating corresponding albums.<br>
`-from-album "GP Album"` Create the album in `immich` and import album's assets.<br>
`-create-albums <bool>`  Controls creation of Google Photos albums in Immich (default TRUE). <br>
`-keep-untitled-albums <bool>` Untitled albums are imported into `immich` with the name of the folder as title (default: FALSE).<br>
`-use-album-folder-as-name <bool>` Use the folder's name instead of the album title (default: FALSE).<br>
`-keep-partner <bool>` Specifies inclusion or exclusion of partner-taken photos (default: TRUE).<br>
`-partner-album "partner's album"` import assets from partner into given album.<br>
`-discard-archived <bool>` don't import archived assets (default: FALSE). <br>

Read [here](docs/google-takeout.md) to understand how Google Photos takeout isn't easy to handle.

### Burst detection
Currently the bursts following this schema are detected:
- xxxxx_BURSTnnn.*
- xxxxx_BURSTnnn_COVER.*
- xxxxx.RAW-01.COVER.jpg and xxxxx.RAW-02.ORIGINAL.dng
- xxxxx.RAW-01.MP.COVER.jpg and xxxxx.RAW-02.ORIGINAL.dng
- xxxxxIMG_xxxxx_BURSTyyyymmddhhmmss.jpg and xxxxxIMG_xxxxx_BURSTyyyymmddhhmmss_COVER.jpg (Huawei Nexus 6P)
- yyyymmdd_hhmmss_xxx.jpg (Samsung)

All images must be taken during the same minute.
The COVER image will be the parent image of the stack

### couple jpg/raw detection
Both images should been taken in the same minute.
The JPG image will be the cover. 

Please open an issue to cover more possibilities.

### Example Usage: uploading a Google photos takeout archive

To illustrate, here's a command importing photos from a Google Photos takeout archive captured between June 1st and June 30th, 2019, while auto-generating albums:

```sh
./immich-go -server=http://mynas:2283 -key=zzV6k65KGLNB9mpGeri9n8Jk1VaNGHSCdoH1dY8jQ upload
-create-albums -google-photos -date=2019-06 ~/Download/takeout-*.zip             
```

## Command `duplicate`

Use this command for analyzing the content of your `immich` server to find any files that share the same file name, the  date of capture, but having different size. 
Before deleting the inferior copies, the system get all albums they belong to, and add the superior copy to them.

### Switches and options:
`-yes` Assume Yes to all questions (default: FALSE).<br> 
`-date` Check only assets have a date of capture in the given range. (default: 1850-01-04,2030-01-01)
`-ignore-tz-errors <bool>` Ignore timezone difference when searching for duplicates (default: FALSE)

### Example Usage: clean the `immich` server after having merged a google photo archive and original files

This command examine the immich server content, remove less quality images, and preserve albums.

```sh
./immich-go -server=http://mynas:2283 -key=zzV6k65KGLNB9mpGeri9n8Jk1VaNGHSCdoH1dY8jQ duplicate -yes
```

## Command `stack`

The possibility to stack images has been introduced with `immich` version 1.83. 
Let use it to group burst  and jpg/raw images together.

### Switches and options:
`-yes` Assume Yes to all questions (default: FALSE).<br> 
`-date` Check only assets have a date of capture in the given range. (default: 1850-01-04,2030-01-01)


## Command `tool`

This command introduce command line tools to manipulate your `immich` server

### Sub command `album delete [regexp]`

This command deletes albums that match with the given pattern

#### Switches 
`-yes` Assume Yes to all questions (default: FALSE).<br> 

#### Example

```sh
./immich-go -server=http://mynas:2283 -key=zzV6k65KGLNB9mpGeri9n8Jk1VaNGHSCdoH1dY8jQ tool album delete \d{4}-\d{2}-\d{2}
```

This command deletes all albums created with de pattern YYYY-MM-DD


# Installation

## Installation from the github release:

Installing `immich-go` is a straightforward process. Visit the [latest release page](https://github.com/simulot/immich-go/releases/latest) and select the binary file compatible with your system:

- Darwin arm-64, x86-64
- Linux arm-64, armv6-64, x86-64
- Windows arm-64, x86-64
- Freebsd arm-64, x86-64

Download the archive corresponding to your OS/Architecture on your machine, and decompress it. 

Open a command windows, go to the directory where immich-go resides, and type the command `immich-go` with mandatory parameters and command.

⚠️ Please note that the linux x86-64 version is the only one tested.


## Installation from sources

For a source-based installation, ensure you have the necessary Go language development tools (https://go.dev/doc/install) in place.
Download the source files or clone the repository. 


# Road map
- [X] binary releases with no dependencies
- [X] check in the photo doesn't exist on the server before uploading
    - [X] but keep files with the same name: ex IMG_0201.jpg if they aren't duplicates
    - [X] some files may have different names (ex IMG_00195.jpg and IMAGE_00195 (1).jpg) and are true duplicates
- [X] replace the server photo, if the file to upload is better.
    - [X] Update any album with the new version of the asset
- [X] delete local file after successful upload (not for import!)
- [X] upload XMP sidecar files 
- [ ] select or exclude assets to upload by
    - [X] date of capture within a date range
    - [ ] type photo / video
    - [ ] name pattern
    - [ ] glob expression like ~/photos/\*/sorted/*.*
    - [ ] size
- [ ] multithreaded 
- [X] import from local folder
    - [X] create albums based on folder
    - [X] create an album with a given name
- [X] import from zip archives without unzipping them
- [X] import google takeout zip archives without unzipping them
- [X] Import Google takeout archive
    - [X] manage multi-zip archives
    - [X] replicate google albums in immich
    - [X] manage duplicates assets inside the archive
    - [X] Use the google takeout date to set the immich date even when there is no exif date in the image.
    - [X] don't upload google file if the server's image is better
    - [X] don't import trashed files
    - [X] don't import failed videos
    - [X] include photos taken by a partner in dedicated album (the partner may also uses immich for her/his own photos)
    - [ ] handle Archives 
- [ ] use tags placed in exif data
    - [ ] JPEG files
    - [ ] MP4 files
    - [ ] HEIC files
    - [ ] name of the file (fall back, any name containing date like Holidays_2022-07-25 21.59)
- [ ] upload from remote folders
    - [ ] ssh
    - [ ] samba
    - [ ] import remote folder
- [ ] Set GPS location for images taken with a GPS-less camera based on
    - [ ] Google location history
    - [ ] KML,GPX track files
- [x] Cleaning different resolution duplicates in the immich server based on their name and date of capture 



# Acknowledgments

Kudos to the Immich team for their stunning project!🤩

This program use following 3rd party libraries:
- github.com/rwcarlsen/goexif to get date of capture from JPEG files
- github.com/ttacon/chalk for having logs nicely colored 
-	github.com/thlib/go-timezone-local for its windows timezone management
-	github.com/yalue/merged_fs v1.2.3 for its FS merging capability
