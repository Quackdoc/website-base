---
title = "Jpeg-xl is kinda cool."
datetime = 2023-11-20T20:42:08Z
tags = ["Images"]
category = "Tech Showcase"
url_name = "jxlkindacool"
---
# Jpeg-xl is kinda cool.
##### It is highly reccomended to read the notes section after reading this page.

While many of the benefits of JXL have been told a lot, there are some things about JXL that I really think should be more popular. JXL is so jam packed full of features that some of the real gems have been ignored. These gems are often features that can be utilized either today, or in a possible short term future.

Note: in this article, when I talk about HDR, I am talking explicitly about the intended "Brightness" of the image. HDR is a poorly, if at all, defined term. However the general use of it seems to be talking about the potential discernible brightness differences within an image. This is how it is used here.

## XYB, more then just compression. Are weird colors a problem of the past?

Did you know that XYB is an LMS based color space? This probably doesn't mean much to a lot of people and that's fine. The super TLDR is that instead of modeling how displays emit light, JXL internally models how humans perceive light. I won't bore you with the complicated technicalities. Nor will I go into how this is "more accurate and future proof". No I will nice and simply talk about the main benefit of this that can be implemented now.

JXL needs to convert this "Human based format" to a "Display based format". However there are a lot of display based formats. Many readers may have heard about sRGB before. This is one such format, however there are many more formats. DCI-P3, Apple-p3 BT.2020 etc. JXL can by necessity when it decodes the image, it can decode to any of these! For those who are a little more technically literate, you may have already realized what this means.

JXL decoders, by necessity, have built in degree color management. If you have a wide gamut monitor and have seen images look wrong on them, obviously wrong, this is typically because the color space of the image is wrong. A good JXL implementation can pretty much make this an issue of the past.

#### Does that mean HDR is easy now?

In short? Sadly No. It's important to note, this format conversion does not take "Tone mapping" into consideration. Tone mapping can be a very complicated topic. However you might be wondering right now what even is tone mapping? Before that, we need to know what a transfer is and why it's important. The first bit of what we need to understand that human's don't see light linearly. Imagine a light bulb. If we have a dim light bulb, and we double the power to it so it double's in brightness, humans don't perceive that as twice as bright. 

Imagine you have a series of numbers, from 0 to 100. Each number represents the intensity of how hard a pixel, or light source is shining. 100 is as bright as it can possibly shine, and 0 is as low as it can possibly go (remember LCD panels don't actually turn off). Humans are much more sensitive to "dimness", so we actually need more data then we physically have to represent the "steps" in this low end. The keen minded may have realized from this, that humans are less sensitive to "brightness" so we can actually take some of that data away.

This is what a transfer function does. It shapes how bright that stepping is so it can better correlate to how humans actually see light. Different transfer functions shape that light in different ways, which allows transfer functions to actually choose how large the range of dimness to brightness it can represent. So, What does tone mapping do? Put simply tone mapping is what allows us to change between greatly different transfer functions while keeping the intended perceived lightness differences of the image.

For a very simple example on why tone mapping is important, [Here is an interactable desmos graph](https://www.desmos.com/calculator/hfybgg2wfc) with sRGB, Gamma 2.2 and Gamma 2.4 as well as PQ transfer curves are graphed. With an image of it directly below. See the Line sticking out like a sore thumb? That's PQ.

![embed an image of the graph](https://files.catbox.moe/1k5q5e.png)

When looking at this graph the Y axis corresponds with the intensity of light with `0` being `off` and `1` being `full intensity`. The X axis corresponds to the actual pixel value as it is actually stored in the file. The bottom parts of each segment are the actual lines, everything else is just math. You can enable and disable the lines by clicking the blue circle on the left side of the math box. It's important to note that with sRGB and Gamma2.2 full brightness is somewhere from 80 to 300 nits. It is supposed to be 80 nits, but there is some variation as to how it's actually done there and it's not uncommon to see them graded with a variety of intended max nits. With PQ however, It has a solid 10000 nit max. The Desmos link has boxes where you can adjust the peak. If you want to see it at an "Intended 1:1" scale, set PQ to "10000", and sRGB for "80". 

This show cases two things, The first being that this is why we cannot display an sRGB directly on a PQ screen, and vice versa. The lines don't get close to corresponding, which means the luminance of the image will be all sorts of messed up. The second showcases why we need tone mapping. If you were to map an sRGB image intended for 80nits 1:1 on a monitor with PQ, where on average "White" will be from 200-300 nits(203 is recommended, but there is some leeway in actual implementations). Chances are you won't even be able to see a good portion of the sRGB image. 

Terminology time. sRGB is a color space. A colour space is defined as a collection of three things. A Transfer, a Gamut, as well as a white point. JXL can handle gamut and white point mapping perfectly fine, but as we covered Transfer is a lot more complicated to map because of the issue where intended peak brightness, or as the proper term is called. luminance of sRGB, and PQ are greatly different, however there is actually another issue then just the luminance range. Gamma2.2 and sRGB's transfer using a *relative* luminance, and systems like PQ and HLG using an *absolute* luminance.

Why does this matter? Well it turns out, that because sRGB is a relative based transfer, It's not actually consistent on *what* an sRGB image is intending it's peak luminance to be. Some images are graded on a 80 nit max display, some on a 200 nit max display etc. It's actually kind of arbitrary. 

Well even pretending we live in a perfect world how do we actually do the tone mapping to make the two systems work together. Lets talk about rendering SDR on HDR for a second. This is commonly known as "Inverse Tone mapping." And this is actually a bit easier to do since we are "expanding" the image instead of "compressing" the image

While "SDR" images can be roughly mapped to `1:203` where `1:` represents the peak brightness of the sRGB image, and `:203` the absolute brightness it will be displayed at. This kind of "dumb" mapping doesn't always look good sadly. But on the other hand, imagine scenario where we do no tone mapping instead, the white image you look at on a normal screen, would be dumping an 10000nit on a theoretical perfect PQ display. To put into context how bright this is. This is about as bright as a high quality office florescent light bulb. Of course, Something that resonates a bit more with would be this scenario. You have one of the new HDR ipads. Open an HDR white image, max out the brightness, go into a room and make it completely dark. This can be harsh enough that people have reported headaches from looking at it too long, and sudden exposure can actually cause small amounts of pain. These devices are about 1000 nits of output.

And this is the *easy* form of tone mapping, HDR to SDR is **significantly** harder to do. We are now compressing the possible brightness differences, but also the maximum brightness of the image needs to be brought down significantly. Being able to reconcile the two issues isn't too hard when we are doing it by hand. We however, aren't doing it by hand, we need a one size fits all glove. This is actually very much non trivial, There are ways to do it, but they do come with a degree of difficulty and performance hit. 

So finally, what *does* JXL do here? JXL does have some tone mapping facilities in it. It can preform some rudimentary tone mapping when decoding the XYB image, However they are quite basic but they do the job when converting SDR to HDR. I however don't recommend it for HDR to SDR since it tends to not do a very good job, Images will usually look kind of messed up.

so yeah, HDR/SDR? not exactly a solved problem. Nothing can easily overcome the limitations of grading an image for a specific luminance range and displaying it on a different range. However make no mistake, even if HDR doesn't become super easy with this, it's still really nice to have proper gamut support baked in. Gamut issues have plagued users long before transfer issues have.

#### OK, The HDR still kinda sucks, but everything else is great right?

Kinda. I know I said that weird colors are a thing of the past, but this is only somewhat true.

A) This only works when the image is XYB encoded. `cjxl` when doing lossless encoding, will not convert the image to an XYB colorspace and tag it as such (even if you grade an image in XYB and encode that with `cjxl`), but will encode the image "as is" instead. While this is good for encoding losslessly, this does sadly mean we don't get the color management benefits of this currently. Now, How much does it matter that you need to use lossy encoding? Even if something isn't lossless, that doesn't always mean the viewers will be able to tell at all. If an image is technically lossy, but it's not expected that a human consumer will be able to tell or notice degradation, these are commonly called "Visually lossless" images. (Though be warned, Lots of companies will take liberties in calling something such)

For example, encoding an image with `cjxl -d0.5 -e 8`, I can't tell the difference between the JXL encodes image and the original image at all. I've attempted multiple blind tests for myself and failed each one. Perhaps if I had a better monitor I could tell the difference, but when I am looking this hard, It's not an issue for delivery images at all. It's also worth noting that when doing this, there is significant space savings to be had as well.

So we have a trade off here. We don't get lossless images, however we can support any kind of image gamut pretty nicely. I would say that is a fair trade off myself at the very least.

B) When doing gamut mapping, it is a little more complicated then just decoding to the gamut. XYB can be converted to XYZ perfectly fine which works great as the middle man. However what happens when you input a BT.2020 color into a XYB, Then decode that to an sRGB image? It will look "mostly right". You see, Gamut mapping is also a little involved, but at least much less complicated then tone mapping. Much like tone mapping. The implementation of gamut mapping seems to be left up to the implementation. However at the very least, even a mediocre gamut map is better then no gamut map. Libjxl does have proper gamut mapping support, so this is not an issue for this specific case, but other implementations may not support it yet.

#### Alright enough advertisement, How do I use it?

A good question, with an easy answer. The first thing to do is make an image with an XYB encoding. While you can use any image, to showcase that this is actually working we are going to take an sRGB image and make it into a BT.2020 image. Encode your image with a lossy setting, if you don't want quality loss you can simply do `cjxl -e 7 -d 0.5 imagein.png imageout.jxl` (if your source is a Jpeg you will need to disable lossless Jpeg, but keep in mind that this may not actually be smaller then the source image `--lossless_jpeg=0`).

The next step is to simply decode the image to a usable format. In this case let's use PNG. To first understand how djxl decodes image colorspace. let's take a look at this example cmdline. `djxl --color_space RGB_D65_SRG_Per_SRG in.jxl sRGB.png`

in.jxl and out.png are simple. but what about the color_space option?, it breaks down like so. 1_2_3_4_5.

1. pixel format. This is the format of the pixels themselves.
2. White point. Part of a colorspace is white point, it defines how "warm" white is.
3. This is the gamut. as we talked about before, this is what we will need to change to support various gamut formats.
4. This is rendering intent. This is information we don't need to care about right now.
5. The last one is transfer. As we talked about before transfer is important because it shapes the light intensity.

So what does this cmdline do? it decodes the image as, RGB pixels, D65 white point, sRGB gamut (also known as BT.709), Perceptual rendering intent, with an sRGB transfer. Modifying this to support BT.2020 is easy enough, although it's not well documented. we can use `djxl --color_space RGB_D65_202_Per_SRG in.jxl 2020.png`. This replaces the SRG gamut with a 202 section which stands for the BT.2020 gamut. We can know open these images in any viewer that doesn't do automatic gamut mapping to compare them and yup! One image is in BT.2020 and another in sRGB. It's nice an easy to tell because in the case for most viewers, BT.2020 will look "wrong". However when we open it up in an app that does do gamut mapping like `mpv`. We can see that the image does look proper still, while `mpv` is reporting a BT.2020 in the gamut.

As for something a little cool, we can also export directly to XYB. This could potentially be useful if you want to run a color managed app. You can directly convert the XYB to whatever format the app you are using is on. `djxl --color_space XYB_Per in.jxl XYB.png`. I have also heard reports that people have been using XYB to squeeze more quality out of Jpeg images. Not something I have personally tinkered with. 

Finally, let's talk a bit about HDR and SDR. As you may have figured it out, we can also export HDR images or SDR images as we please. As said above, I only recommend doing from SDR to HDR. We can export it like so `djxl --color_space RGB_D65_202_Per_PeQ --display_nits=1000 in.jxl HDR.png` or we can use `djxl --color_space RGB_D65_202_Per_HLG --display_nits=100 in.jxl HDR.png`. There is a non insignificant chance you will encounter some visible brightness changes. However you will generally get close enough that it will be fine.

## Hey, You said that HDR is hard? But I heard about this new tech called "Gain maps", what about that?

So while I just talked about how luminance tone mapping is actually pretty complicated (particularly when dealing with HDR to SDR), there is however something that is promising to make it much easier for us. Gain maps are an additional image layer that can be used to add extra luminance information to SDR images, effectively allowing us to create both an HDR image and an SDR image at the same time, in the same file, with minimal overhead.

These aren't quite as good as native HDR images, they do have limitations. Despite this      users who consume these images still get the "HDR" benefit and a lot of consumers find this to be quite good. On the other hand, people with a traditional SDR display don't have to worry about the complicated tone mapping mentioned previously, They will still receive the SDR picture as the original artist intended it. Whether this be Digital artist or a photographer, Both SDR and HDR users can enjoy a first class experience. 

So after all this, does JXL support gain maps? Well no? Gain maps aren't a part of the JXL spec, but they also aren't a part of JPEG's spec either. Readers may be pleased to hear that JXL allows a large number of layers. This means JXL can "natively support" encoding gain map images, and while there is no spec for this the potential for this use case is there. It's up to ecosystem to decide whether or not this will be a route they will go.

## I have a need for speed that can't be satiated, but oh boy does JXL get close.

Decode speed of lossy JXL images is actually really fast. Ok, yeah sure, even a potato can decode a single image. However lets say we have 30 images on screen at once. "Who needs that?" one may ask. Galleries of course! There are hundreds of galleries on the web, in our phones, on our PCs. Ok, well JXL can be useful here for sure. But when I say I have a need for speed, **I mean real speed**.

Something I have been testing (although because of FFmpeg updates, this is now broken, woes of unofficial patches,) Is using JXL is a mezzanine format. A very quick TLDR on what a mezzanine format is, it's a video format comprised of a sequence of still photos designed to make video editing, well not suck. One needs only three major things from a mezzanine.

1. still images because that makes frame perfect seeking instant. 
2. fast enough decode speed to make scrubbing smooth enough. 
3. Consistently high quality.

Does JXL do these well? for still images, Obviously yes, we wouldn't be hear right now if this was a no. As for consistently high quality, JXL can do this quite nicely on the higher effort levels for sure.

So, how fast *is* jxl?

* 30fps? Beginner driver or elderly driver? can't tell.
* 60fps? Maybe you can enter turtle races. (Lossless JXL can be found here. Magically faster then PNG sequences for me somehow... I think Libjxl devs sold their souls to the devil.)
* 90fps+ Ah, here you are JXL!

Yeah, that's pretty fast, "But Quack! CineForm is much faster then that! And ProRes is even faster!!!" Okay yeah that is true. I lost on this battle, but they are also about 2x the size for similar quality. If you run out of SSD space for your mezzanines, it has happened to me twice, and I swear it will not again. (I said this the first time too.) You will learn the true meaning of slow. 

I'm not lying either, this was taken from march of this year (2023) 98FPS with `mpv` under Wayland using Vulkan + mailbox. I was getting from 90-120FPS when I was trying this! ![marine-unison](https://cdn.discordapp.com/attachments/549650852839686153/1087020829969227926/image.png)

"OK quack, That's cool and all, but not everyone has a good PC, and I don't really care about a video, I just want to make sure my old craptop will load them fast enough. What about Mr. Low end? No no, Not you old laptop low? I mean **low** end"

Sure, you want low end? do you remember those old windows netbooks? I do, in fact I have one right here!. 

A HP compaq mini with the specs as follows

```  
root@archiso ~ # neofetch --backend off
root@archiso
------------
OS: Arch Linux i686
Host: Compaq Mini 110c-1100 0394110000001C00000300000
Kernel: 6.1.10-arch1-1.0
Uptime: 29 mins
Packages: 398 (pacman)
Shell: zsh 5.9
Resolution: 1024x600
Terminal: /dev/pts/0
CPU: Intel Atom N270 (2) @ 1.600GHz
GPU: Intel Mobile 945GM/GMS/GME, 943/940GML Express
Memory: 215MiB / 977MiB
 
```

Now, lets see how fast libjxl can be.
The first test is a 4k JXL image encoded from a JPEG image using `cjxl -d 0 -e 7 --lossless_jpeg=0`

if you have a JXL enabled browser, the images will embed for you! If not, the links will be at the bottom of this section

```
root@archiso ~ # hyperfine --runs 5 'djxl ina4k-nore.jxl --disable_output'
Benchmark 1: djxl ina4k-nore.jxl --disable_output
  Time (mean ± σ):     19.803 s ±  0.072 s    [User: 38.257 s, System: 0.654 s]
  Range (min … max):   19.727 s … 19.882 s    5 runs

hyperfine --runs 5 'djxl ina4k-nore.jxl --disable_output'  192.25s user 3.80s system 195% cpu 1:40.10 total
```
![<JXL> Ina is a comfy streamer, A shame you can't see this image.](https://files.catbox.moe/5obp2y.jxl)

For this next image, This is "The Woag". It is an 8k image lossless encode using `cjxl -e 7 -d 0` with Jpeg reconstruction. This image might even slow down a desktop browser!

```  
root@archiso ~ # hyperfine --runs 5 'djxl thewoag.jxl --disable_output'
Benchmark 1: djxl thewoag.jxl --disable_output
  Time (mean ± σ):      8.871 s ±  0.017 s    [User: 15.441 s, System: 1.623 s]
  Range (min … max):    8.841 s …  8.883 s    5 runs
  
hyperfine --runs 5 'djxl thewoag.jxl --disable_output'  78.02s user 8.58s system 190% cpu 45.428 total
```
![<JXL> The woag is pretty big! This might be slowing down JXL enabled browsers though.](https://files.catbox.moe/fsmdyy.jxl)

And finally we have an image encoded with `cjxl -e 7 -d 1` from a PNG source.
```
root@archiso ~ # hyperfine --runs 5 'djxl mona.jxl --disable_output'
Benchmark 1: djxl mona.jxl --disable_output
  Time (mean ± σ):      5.944 s ±  0.007 s    [User: 10.453 s, System: 0.925 s]
  Range (min … max):    5.934 s …  5.954 s    5 runs

hyperfine --runs 5 'djxl mona.jxl --disable_output'  53.07s user 5.03s system 188% cpu 30.790 total
```

![<JXL> I may have a small addiction to Genshin, The game is great and so is the art you are missing!](https://files.catbox.moe/6o1t4r.jxl)

As you can see, Even on a really old 1GB dual core atom, JXL is surprisingly fast. I wouldn't recommend browsing a JXL enabled page. But I also wouldn't reccomend using one of these machines in the first place. It is also worth noting that this is an Arch linux Live boot, not a full install.

Ina: https://files.catbox.moe/5obp2y.jxl

Thewoag: https://files.catbox.moe/fsmdyy.jxl

Mona: https://files.catbox.moe/6o1t4r.jxl

## Can you tell me how consistent existing JXL encoders are? You mentioned it before.

Yeah sure. So there are two methods we can use for encoding the image sequence, the easy method for this would be to use FFmpeg for this. While this is a valid method, I want to use a bit of a slower effort, and FFmpeg isn't great at parallelizing this. Instead, I will use FFmpeg to decode the image sequence into a series of PNG images. I am mainly doing this because it's fast enough, and using different formats makes this just a wee bit easier on me. The encode I will run is `ffmpeg -i video.mp4 -c:v png img-seq\frame-%04d.png`

The next step is to encode each frame into JXL, for this Linux users can use `parallel -j NUM cjxl -d 0.5 -e 6 {} {.}.jxl` where NUM is the number of parallel threads. On windows you can use powershell like so `ForEach -Parallel`, or you can use a tool like `rust-parallel` which is a windows compatible parallel command execution tool similar to parallel. To use rust-parallel you can use `ls.exe | rust-parallel -p -j 4 -r '(.*).(png)' cjxl -e 6 -d 0.5 {0} {1}.jxl`

It's easier to move the JXL and PNG images into their own folders so do that now. I will then decode the JXL images back to png, because the tool I will be using is `ssimulacra2rs` and it uses vapoursynth for input. This makes it great for doing per frame testing, however the most of the common input tools on windows don't support JXL yet, if you are on linux. you may be able to skip this step as the tools may support JXL. On windows I can use `ls.exe | rust-parallel -p -j 4 -r '(.*).(jxl)' djxl {0} {1}.png` to convert the frames back to PNG.

My main folder now has two sub folders. `jxl/frame-%04d.png` and `png/frame-%04d.png` where the png folder is the source, and the jxl folder is the encoded ones.

Next we create two vpy files that we can use them as inputs. This allows us to compare them as an image sequence instead of indivually, this is a bit faster and lets us graph the output. create one file for JXL folder, and one file for PNG folder. the JXL folder is given below

```
import vapoursynth as vs
core = vs.core
clip = core.lsmas.LWLibavSource(source='jxl/frame-%04d.png')
clip = core.resize.Bicubic(clip=clip, format=vs.YUV444P10, matrix_s="709")
clip.set_output(0)
```

Finally execute this command to run the comparison program. `ssimulacra2_rs.exe video -f NUM_THREADS -g png.vpy jxl.vpy`. `ssimulacra2rs` is quite slow, however it is quite an accurate method of comparing images/videos to one another. As we can with the graph below, JXL manages to exceed 90 ssimu2 with ease.

![The output graph for JXL](https://files.catbox.moe/lzn4uv.png)

How does this compare with prores? Good question using prores hq, these were the scores I got.

![Prores is not as stable, but more then good enough still!](https://files.catbox.moe/gm7dhn.png)

Prores is less consistent but most frames scored better. However anything past a 90 is usually good enough for a mezannine. But how does file size compare? JXL: `58.99 MiB` Prores: `237.70 MiB` As we can see, JXL is less then half the size! We could probably use some of that savings to bump the quality some, but for me it simply isn't necessary.

## OK, JXL is kinda cool. I would use it but man, C++ sucks.
I hear you loud and clear. If it was any louder I might go deaf. I know the pain, and I myself am not the biggest fan either. However [have no fear, Rust is here!](https://github.com/tirr-c/jxl-oxide) Well for decoding at least. But still! jxl-oxide is a third party, native rust decoder, and it is pretty fast. The important part of jxl-oxide however, Is not the MIT license, It's not that rust is the best programming language ever. It's this, 

```
➜  jxl-oxide git:(main) RUSTFLAGS='-C code-model=small -C debuginfo=none -C opt-level=z -C strip=symbols -C codegen-units=1 -C panic=abort -C target-feature=+crt-static' cargo build --target=x86_64-unknown-linux-musl --release
--SNIP--
➜  jxl-oxide git:(main) ldd ./target/x86_64-unknown-linux-musl/release/jxl-dec
        statically linked
➜  jxl-oxide git:(main) ls -llh ./target/x86_64-unknown-linux-musl/release/jxl-dec
-rwxr-xr-x 2 quack quack 3.4M Nov 19 23:11 ./target/x86_64-unknown-linux-musl/release/jxl-dec
➜  jxl-oxide git:(main) upx --lzma --best ./target/x86_64-unknown-linux-musl/release/jxl-dec
--SNIP--
➜  jxl-oxide git:(main) ls -llh ./target/x86_64-unknown-linux-musl/release/jxl-dec
-rwxr-xr-x 1 quack quack 966K Nov 19 23:11 ./target/x86_64-unknown-linux-musl/release/jxl-dec
```

That's right, with a simple cargo build command line, we can make a 3.4M binary to decode to either PNG or npy, with proper static compilation so it will run on almost any Linux system, and when we use UPX to compress it, it's a mere 966K. No need to muck about dependencies. No need for weird cross compile shenanigans and these sizes are without rebuilding std.

Thanks to being rust, this also means you can easily use jxl-oxide crate in your projects. There is even a usable [wasm demo](https://gitlab.com/nitroxis/jxl-wasm) too for people who would want to use it targeting that. (I didn't include it on this page due to the 16k woag image. JXL might be fast, but no matter what wasm has it's limitations.)

Of course, one should keep in mind that jxl-dec binary does too have some limitations, jxl-dec It only supports PNG and npy output. You can't specify output colorspace either at the moment. 

But how does it preform on that crappy laptop from before? Well not great, Jxl-oxide works fine on newer systems, but when you load it on this crapy laptop...

```
130 root@archiso ~ # hyperfine --runs 2 './jxl-dec mona.jxl'
Benchmark 1: ./jxl-dec mona.jxl
  Time (mean ± σ):     168.351 s ±  0.236 s    [User: 280.878 s, System: 3.043 s]
  Range (min … max):   168.185 s … 168.518 s    2 runs

hyperfine --runs 2 './jxl-dec mona.jxl'  563.19s user 7.00s system 168% cpu 5:37.72 total
```

Ouch. The results speak for themselves. However it's worth noting that this 32bit binary, is build without even so much as SSE suport which hurts a lot. That ram limitation also hits really hard, Keep in mind that this image is `5221x2847`. Make no mistake however, JXL-oxide is still a competent decoder for any somewhat modern machine that has remotely close to a reasonable amount of ram. Of course, there are always optimizations to be made.

```
hyperfine --runs 10 'djxl mona.jxl --disable_output' './target/release/jxl-dec mona.jxl'
Benchmark 1: djxl mona.jxl --disable_output
  Time (mean ± σ):     200.3 ms ±  28.0 ms    [User: 1348.1 ms, System: 364.4 ms]
  Range (min … max):   175.0 ms … 259.3 ms    10 runs

Benchmark 2: ./target/release/jxl-dec mona.jxl
  Time (mean ± σ):     561.8 ms ±  23.2 ms    [User: 1954.7 ms, System: 795.2 ms]
  Range (min … max):   515.5 ms … 592.4 ms    10 runs

Summary
  djxl mona.jxl --disable_output ran
    2.80 ± 0.41 times faster than ./target/release/jxl-dec mona.jxl
```

This was tested on a arch WSL2 instance on my Ryzen 2600. While yes, jxl-oxide is slower, it's certainly no slouch either. It is still well within reasonable speeds. Just make sure to either compile it with native optimizations, or simply spring for a 64bit PC instead.

## A great colour space, Easy decoder, Amazing flexibility and speed to boot! What am I loosing here chief?

Well, Support, JXL is still a new format, at the time of writing, On android, you wont find it any android gallery app that I know off, only a couple apps actually go out of their way to support it like the popular manga app `tachiyomi`. Since android doesn't support JXL there is little support here. Hopefully one day we might see official platform support here.

You also wont find support in very many web browsers either with Chrome removing support, and Firefox being Firefox and ignoring the issue outright. However thanks to the community, there are many forks available of both browsers that have good support. I currently use `Thorium` and `Waterfox` on desktop, and `Cromite` on android. Also Webkit based browsers like `Safari` and `Gnome web` also support JXL officially now thanks to apple pushing the ecosystem. I guess apple does what Firefox doesn't.

Linux support is great. Keep in mind however that people in general have a misconception about how Linux handles these things. Linux relies on the application themselves to support nearly everything. There is no generic platform support for these. There are just... a lot of platforms to choose from some more generic then others.  Thankfully however, independent app ecosystems are generally pretty good, both QT and GTK support JXL, dedicated tooling like imagemagick and FFmpeg support it too. So you shouldn't have too many issues here.

As alluded to above, Apple is doing great here. They are offering platform support for JXL now so you should have zero issues here soon enough as long as you are up to date on any of the apple ecosystem devices.

As for windows, there is a plugin to add it to windows imaging component. It works ok, it's pretty limited but its... fine. It of course is not official support. Hopefully soon windows will decide to officially support JXL.

You can help push adoption by recommending it to people who can use it, and asking your applications you use often to support it. Websites are already serving JXL too, and some like this page you are viewing won't even work 100% properly without JXL support. So you can report those websites as broken. 

If you run a website, You can start serving JXL today. Many CDNs support JXL for customers, If you aren't serving a large amount of images on a single page, you always have the option to implement a WASM decoder. I have run into an issue that despite JXL decoding fast enough, image rendering will actually be paused while another image is decoding. If anyone knows how to prevent chrome and Firefox from doing this, send me a message on mastodon or perhaps even make a PR [at this repo](https://github.com/Quackdoc/jxl-wasm). I would love to get that working and enable it here.


## Notes.
#### Color

In this article I took some liberties with the terminology and explanations to keep it simple for people who may have not come across these terms and don't have great knowledge into the subject matter. This was particularly done when talking about color. For readers who wish to learn about the proper terminology, explanations, and gain a better in-depth knowledge into color. I highly recommend reading into these sources

[Hitchhikers guide to digital color.](https://hg2dc.com/) The language is crass but the knowledge is first class for an introduction into color.

[This is a git repo](https://gitlab.freedesktop.org/pq/color-and-hdr/)  containing a large variety of information for color and HDR information intended for developers, The knowledge at this time isn't well structured, but it is a great repository of info.

[Bruce Lindbloom's website](http://www.brucelindbloom.com/) has some of the best knowledge when it comes to some of the math behind the core concepts of working with color.

[Cinematic color](https://cinematiccolor.org/) talks about color management in the context of media production.

[A PDF](https://graphics.stanford.edu/courses/cs148-10-summer/docs/2010--kerr--cie_xyz.pdf) about the history and some technical aspects of the XYZ and xyY colorspaces. A significant colorspace when it comes to detailing how human vision works and conversions between formats.

#### Rust vs C++

Ok, No I don't actually *hate* C++, However some developers of various projects have complained about pulling libjxl into their build chain before and it not playing nice. For some projects, jxl-oxide might indeed be significantly easier to pull into their build chain, but as it stands, I don't believe jxl-oxide can actively be built as a library and directly integrated with other languages.
