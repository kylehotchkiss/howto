## Concatenate (join) GoPro Videos

GoPros split video files every 4 gigabytes. This is likely due to a filesystem limitation. It's easy to join them using the command line.

1) Install FFMpeg 
```
brew install ffmpeg
```
2) Copy all desired videos to join to a folder on your desktop
3) Open that folder in your command line
4) `ls` all the video files
5) Create an `input.txt` file. You'll need to insert each video filename into the file on its own line, with `file '%'` surrounding it.
File should look like this:

```
file 'GOPR0806.MP4'
file 'GP010806.MP4'
file 'GP020806.MP4'
file 'GP030806.MP4'
file 'GP040806.MP4'
file 'GP050806.MP4'
file 'GP060806.MP4'
```
6) Run ffmpeg to concatenate:

```
ffmpeg -f concat -i input.txt -c copy output.mp4
```

The output file should match the codecs and containers as the source files.

7) Pass video through Handbrake to reduce the size a bit before uploading or sharing.

### Reference 
1) https://trac.ffmpeg.org/wiki/Concatenate#samecodec
2) https://handbrake.fr/
