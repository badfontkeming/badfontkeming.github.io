# How to **Actually** render transparent video (i.e. with an alpha channel) from Sony/Magix Vegas Pro

_useful for exporting transparent or semitransparent videos into streaming software such as OBS and XSplit, as well as for use in web pages and other "finalized" usecases_

_i know it hasn't been called sony vegas since version 13 but you'll take my old habits from my cold dead hands_

## Why this guide is different

**There's one very important thing this guide covers that other guides don't.** The color of transparent video that Vegas exports actually gets blended with black based on how transparent each pixel is, which makes it display incorrectly in environments like OBS, XSplit, and web browsers. More on this below in the section "Dealing with Premultiplication".

**This guide also offers a completely different method of exporting the video itself.** Most guides I've seen for getting transparent video out of Vegas revolve around one of the four approaches, all of which I wasn't satisfied with:

- Exporting uncompressed .avi files or PNG sequences (tons of disk space used, potential stress on an SSD if you do this often due to amount of bytes written)
- Using an outdated and vulnerable version of Quicktime to encode compressed transparent video (seriously? no)
- Using a paid third party extension which can render video with external codecs (I paid $15 for this editor in a humble bundle. I'm not paying more than I paid for the editor just so I can actually use it.)
- Using a frameserver to feed the video into an encoder that can handle transparent videos (probably the best of these options, but it's clunky, the only free frameserver isn't under active maintenance, and I wasn't able to get the frameserver to play nice with FFmpeg, my preferred encoder)
- _Bonus: hoping a newer version of Vegas can/will do this (i can't find info on this and am not spending $100+ to find out)_

Instead of these four approaches, this guide covers my approach: 

- Render two videos: one is the original colors, and the other is an alpha mask. Blend these together in FFmpeg to produce a transparent .webm (or other container). This gives us the ability to fix the color issues mentioned above at the same time as well.

## Dealing with Premultiplication

As mentioned before, Vegas distorts the colors of transparent video by blending them with black. If you're not concerned with the technical explanation of why this is a thing, the instructions in the "How It's Done" section will give you the fix.

When dealing with transparency, there are essentially two ways of handling color when dealing with an alpha channel. **Straight** alpha refers to complete independence between the color and alpha channels. This is how most image editors export transparent images, and so long as everything along the toolchain supports straight alpha, it's generally the best way to do things. **Premultiplied** color refers to compressing the color range of any given pixel in an image down to the transparency of that pixel, and it's the only form of color you can get out of Vegas as far as I'm aware. It causes videos and images to look "right" in environments that don't support alpha channels (remember how weird transparent PNGs would look when you uploaded them to Twitter?), at the cost of requiring compensation to look right in environments that _do_ support alpha. In essence, the color is being "multiplied" by some background color (in this case, black).

FFmpeg basically offers us a way to "convert" the colors from premultiplied back to straight. While the color detail will be crushed, the detail loss won't show up when the alpha channel is in play, since alpha blending ends up equally "re-crushing" those colors anyway. **This only works if you don't tamper with the alpha channel after exporting. Re-export your video if you need to change the alpha channel.**

### Comparison Shots

Here's a visual example of what I'm talking about. Not compensating for premultiplied colors on transparent videos and images results in a loss of color detail and the illusion of the image being even more transparent than it actually is.

(TODO: image or interactive demonstration of the effects of uncompensated premultiplication.)

## How It's Done

(todo: this is going to need a metric ton of screenshots)

(todo turn outline into actual instructions)

export color video

use Mask Generator as a Video Output FX in order to export an alpha mask

**make sure no chroma subsampling is enabled on your exports or else things will look substantially worse. use an overkill bitrate since you're about to double-encode this video and generational loss is stronger than you think in modern codecs**

ffmpeg command line. this works in windows and linux (you're using vegas why are you doing this in linux), make sure you have the latest statically compiled build as the version in many distros' repos is out of date enough that VP9 encoding didn't yet have a transparent pixel format. the big mic-drop right here is that FFmpeg has the 'unpremultiply' filter, which does exactly what we want it to do. nice. gotta love the ffmpeg devs, it's a hard tool to learn but it really does come in clutch for things like this.

```powershell
./ffmpeg -i .\colorinput.mp4 -i .\maskinput.mp4 -filter_complex "[0:v][1:v]alphamerge,unpremultiply=inplace=1" -r YOUR_FRAMERATE -b:a YOUR_AUDIO_BITRATE -b:v YOUR_VIDEO_BITRATE -c:v vp9 -pix_fmt yuva420p -auto-alt-ref 0 transparentvideo.webm
```

the container of the video is implicitly taken from the extension of the output file. we're using webm here because it's the most popular container that supports transparency, and it's supported by OBS, xsplit, and all browsers. any other container that fits all those can be used if you feel like it, i guess. if for some reason you find something that doeesn't support vp9, switch it to vp8 and you'll probably be fine (same pixel format is ok).

i dont know why `-auto-alt-ref 0` is needed. it satisfies the small tiny little warlock inside your computer. be nice to him. your life depends on it. 