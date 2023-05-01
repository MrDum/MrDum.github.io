---
title:  "Convert a movie file to x265 with ffmpeg"
image:
  thumbnail: "/assets/images/banner.jpg"w
  path: "/assets/images/banner.jpg"

header:
  teaser: "markup-syntax-highlighting-teaser.jpg"
tags:
  - ffmpeg
---


# 1. Convert a movie file to x265 with ffmpeg built into Ubuntu

apt-get -y install ffmpeg is enough on Ubuntu 18.04 to convert a video to h.265. I use the following script, which stores converted file in the folder above the one you are currently in:



```console
#!/bin/bash
for i in *.mkv;
  do name=${i%.*}
  ffmpeg -i "$i" -c:v libx265 -crf 24 -preset veryslow -c:a copy "../${name}.x265.mkv"
done
```

The official ffmpeg wiki suggests a crf quality value of 28, but I found that to be too low for the 720p material I converted and increased (!) the quality to 24. The preset determines the size of the output file, so you pretty much have to choose veryslow, if you want a small file. The audio is merely copied – we will get to that in the second step. Note that you may have to adjust the script to account for subtitles (make sure they are copied over correctly).
# 2. Compile ffmpeg with libfdk_aac (Fraunhofer AAC codec) and then convert the audio of a movie file with it

Due to a license conflict, FFmpeg binaries cannot be distributed with libfdk_aac and end users have to compile it manually. Fortunately, there are scripts that do the work for us. The github user rafaelbiriba created one for Ubuntu 18.04 (forked by chrisvaughn).

```console
wget https://gist.githubusercontent.com/rafaelbiriba/7f2d7c6f6c3d6ae2a5cb/raw/2b760600a39c560df19c0c6140213163a556d9b1/install_ffmpeg_libfdkaac.sh
chmod +x install_ffmpeg_libfdkaac.sh
apt-get -y install screen
screen
./install_ffmpeg_libfdkaac.sh
(detach from the screen session with ctrl+a+d and return to it with ctrl+r, quit it with “exit”)
```

The compiled ffmpeg binary can be found in the folder bin in your current directory. It took about 15 minutes for me to compile on a single core. Adjust the path to the binary in the following script, which only converts the audio of a movie file and stores the converted file in the parent folder. Again, make sure to adjust the script for subtitles.

```console
#!/bin/bash
for i in *.mkv;
  do name=${i%.*}
  /root/bin/ffmpeg -i "$i" -c:v copy -c:a libfdk_aac -vbr 5 -cutoff 18000 "../${name}.AAC.mkv"
done
```

I chose the highest variable bitrate setting available (-vbr 5), which came out to be about 100 kbit/s at stereo audio. There will be a warning claiming “Note, the VBR setting is unsupported and only works with some parameter combinations”. The user nu774 of the hydrogenaudio forum tested which parameters allow what set of VBR quality parameters. I did not use HE-AAC or HE-AACv2 (the latter supports only stereo) because they do not support a variable bitrate quality of 5 for as many sample rates as LC-AAC and the further file size reduction would be miniscule.
-cutoff 18000 is necessary because it would otherwise cut off the frequency at 14,000Hz (a human can hear up to 20,000Hz)
See also the official ffmpeg wiki. 

[source](https://tech.tiq.cc/2020/04/how-to-convert-videos-to-h-265-hevc-video-with-fdk-aac-audio-on-ubuntu-and-debian-linux-x265-and-libfdk_aac/)