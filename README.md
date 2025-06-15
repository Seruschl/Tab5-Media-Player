# Tab5-Media-Player
Play video file on M5Stack Tab5

## How to use

You can play AVI video files encoded in MJPEG+MP3 that are stored on USB storage/SD card.
USB and SD card are recognized as connected at startup. If both are connected, USB will be mounted first.
Since SD card slots are slow and often fail to load data in time, it is recommended to use USB.

### How to convert videos

Use [FFmpeg](https://ffmpeg.org), which can be installed from a package manager such as Homebrew or downloaded from the official site.

- For 16:9 video files with a resolution of 1280x720 or higher
  - ```
    ffmpeg -i input.mp4 \
        -vf "transpose=2,scale=720:1280,fps=30" \
        -c:v mjpeg -pix_fmt yuv420p -q:v 15 \
        -c:a mp3 -ac 2 \
        -f avi output.avi
    ```
  - Description of each option
    - `-i input.mp4`
      - Please specify the source file to be converted
    - `-vf "transpose=2,scale=720:1280,fps=30"`
      - Rotate video 90 degrees, scale to 720x1280, and convert frame rate to 30 fps
      - Encoding with the image pre-rotated to 720x1280 eliminates the need for rotation processing when playing back on the M5Stack Tab5 side, resulting in stable playback.
    - `-c:v mjpeg -pix_fmt yuv420p -q:v 15`
      - mjpeg format, yuv420 color space, video compressed at 15 quality
      - The `-c:v` and `-pix_fmt` values are fixed and no other video files can be played.
      - The value of `-q:v` is 2~31, the smaller the value, the higher the image quality and the larger the file size.
    - `-c:a mp3 -ac 2`
      - Compress audio in 2-channel stereo mp3
    - `-f avi output.avi`
      - The destination file format is fixed to avi, and the file name should be changed to any name.
- For video files with a resolution of 1280x720 or lower and a frame rate of 30 fps or lower
  - ```
    ffmpeg -i input.mp4 \
        -c:v mjpeg -pix_fmt yuv420p -q:v 15 \
        -c:a mp3 -ac 2 \
        -f avi output.avi
    ```
  - If the resolution is small, M5Stack Tab5 will automatically do the enlargement/rotation process, so the `-vf` option is not necessary.
  
### If the video does not play properly

- Sound is choppy and plays slowly.
  - If you are using an SD card slot, use a card reader or similar device to connect via USB, or change the following options during video conversion to reduce the file size
    - `scale=720:1280` : Decrease the value to reduce the resolution of the video.
    - `fps=30` : Decrease the value to lower the frame rate. If the frame rate is not an integer value such as 29.97, make it an integer value such as 30.
    - `-q:v 15` : Increase value and decrease quality
- frame drop
  - If you try to play a video such as 60fps and MJPEG decoding and drawing is not completed in time, `E (11528) AVIPlayer: Frame dropped.` will be output to JTAG/UART and frame drop will occur.
  - Since the data that cannot be rendered is wasted, we recommend converting up to about 45 fps for 720p.

### Video Conversion Adjustments

The best option depends on the content you are playing and the speed of your storage. You may be able to get a higher quality/higher frame rate by doing the following

- Increase the value of `-q:v` to a level where it is likely to be read out in time
- Smoothing output rate with `-b:v` option
- Use an encoder that can compress JPEGs with good quality, such as mozjpeg
