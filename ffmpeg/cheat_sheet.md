# FFMPEG cheat sheet

Convert .mov screen-records from macOS to .mp4 videos for the web, while lowering frame rate to 24 fps.

```bash
ffmpeg -i input.mov -r 24 output.mp4
```

ffmpeg does a great job at lowering the file size. With this command above, a 14mo mov file is turned into a 1,8mo mp4 file, without much quality loss.



I have a retina display so the records have a high resolution. I like to scale down the video resolution by 2 to reduce the file size.
However sometimes the dimensions are not divisible by 2. So in case we need to crop that 1px before we compute the new resolution.

```bash
ffmpeg -i input.mov -r 24 -vf "scale=iw/2:ih/2,crop=trunc(iw/2)*2:trunc(ih/2)*2" video.mp4
```



## Other cheat sheet

[Other cheat sheet](https://github.com/jamesplease/ffmpeg-cheatsheet)

