---
title = "Jpeg-xl is kinda cool."
datetime = 2023-11-20T20:42:08Z
tags = ["Images"]
category = "Tech Showcase"
url_name = "jxlkindacool"
---
# Jpeg-xl is kinda cool.

While many of the benefits of JXL have been told a lot, there are some things about JXL that I really think should be more popular. JXL is so jam packed full of features that some of the real gems have been ignored. These gems are often features that can be utilized either today, or in a possible short term future.

Note: in this article, when I talk about HDR, I am talking explicitly about the intended "Brightness" of the image. HDR is a poorly, if at all, described term. However the general use of it seems to be talking about the potential brightness differences in an image. This is how it is used here.

## XYB, more then just compression. Are weird colors a problem of the past?

Did you know that XYB is an LMS based color space? This probably doesn't mean much to a lot of people and that's fine. The super TLDR is that instead of modeling how displays emit light, JXL internally models how humans perceive light. I won't bore you with the complicated technicalities. Nor will I go into how this is "more accurate and future proof". No I will nice and simply talk about the main benefit of this that can be implemented now.

JXL decoders, by necessity, have built in degree color management. If you have a wide gamut monitor and have seen images look wrong on them, obviously wrong, this is typically because the color space of the image is wrong. This can pretty much make this an issue of the past.

#### Does that mean HDR is easy now?

In short? No. It's important to note, this does not take tone mapping into consideration. Tone mapping can be a very complicated topic. For a very simple example on why tone mapping is important, [The link here](https://www.desmos.com/calculator/hfybgg2wfc) is a Desmos graph with sRGB, Gamma 2.2 and 2.4 as well as PQ transfer curves are graphed. You can roughly take a look at these graphs, and the lines correspond with the intensity of light with `0` being `off` and `1` being `full brightness`. The bottom parts of each segment are the actual lines, everything else is just math. You can enable and disable the lines by clicking the blue circle on the left side of the math box. It's important to note that with sRGB and Gamma2.2 full brightness is intended to be somewhere from 80 to 200 nits. It's suppose to be 80 nits, but there is some leeway there. with PQ however, It's intended to be be 10000 nits. The Desmos link has boxes where you can adjust the peak. If you want to see it at an "Intended 1:1" scale, set PQ to 10000 nits, and sRGB for 80 nits. Chances are you won't even be able to see sRGB.

sRGB is a color space. A colour space is defined as a collection of three things. A Transfer, a Gamut, as well as a white point. JXL can handle gamut and white point mapping, however Transfer which represents the intensity of the pixel, and this is a lot more complicated to do. This is due to Gamma2.2 and sRGB's transfer using a *relative* luminance, and systems like PQ and HLG using an *absolute* luminance.

Without getting to into the specifics, you need *some kind* of tone mapping to make the two systems work together. While "SDR" images can be roughly mapped to `1:203` where `1:` represents the peak brightness of the sRGB image, and `:203` the absolute brightness it will be displayed at. One will often find that these don't always look good. For instance, without tone mapping, the white image you look at on a normal screen, would be dumping an 10000nit on a theoretical perfect PQ display. For reference the new Ipad Pros have a 1000nit display.

1000 nits is bright enough that at night time, people have reported developing headaches from looking at it. JXL does support "some tone mapping" but the tone mapping implementation seems to be left up to the decoder/implementation itself. Libjxl does have some support for tone mapping, It seems to work... OK. Not super great but not super bad.

But make no mistake, Even if HDR doesn't become super easy with this, it's still really nice to have proper gamut support baked in. Gamut issues have plagued users long before transfer issues have. 

#### OK, The HDR still kinda sucks, but everything else is great right?

Kinda. I know I said that weird colors are a thing of the past, but this is only somewhat true.

A) This only works when the image is XYB encoded. Libjxl when doing lossless encoding, will not convert the image to an XYB colorspace, but will encode the image as is. While this is good for encoding losslessly, this does sadly mean we don't get the color management benefits of this. That's right, JXL only has that great color management with lossy encoding. Now, How much that will actually matter is a different story. Even if something isn't lossless, that doesn't always mean the viewers will be able to tell at all. Even if an image is lossy, These can still be "Visually lossless" Meaning that it could be hard, if at all possible, to tell the difference between the source or and the encode when using our eyes, even when we blow the image up to insane degrees.

Using ssimulacra2 as a guide line an image encoded with `cjxl -d0.5 -e 8` image will get a score of `91.7` and indeed, I can't tell the difference between the JXL encodes image and the original image at all. I've attempted multiple blind tests for myself and failed each one. Perhaps if I had a better monitor I could tell the difference, but when I am looking this hard, It's not an issue for delivery images at all.

So we have a trade off here. We don't get lossless images, however we can support any kind of image gamut pretty nicely. I would say that is a fair trade off myself at the very least.

B) When doing gamut mapping, it is a little more complicated then just decoding to the gamut. XYB can be converted to XYZ perfectly fine. but what happens when you input a BT.2020 color into a XYB, Then decode that to an sRGB image? It will look "mostly right". You see, Gamut mapping is also a little complicated. Much like tone mapping. The implementation of gamut mapping seems to be left up to the implementation. However at the very least, even a mediocre gamut map is better then no gamut map. 

## Support for HDR Gain maps? HDR and SDR that doesn't suck with a single image?

So while I just talked about how luminance tone mapping is actually pretty complicated (particularly when dealing with SDR to HDR), there is however something that will be making it much easier for us. This is gain maps. Gain maps are an additional image layer that can be used to add extra luminance information to SDR images, effectively allowing an HDR rendition of an SDR image. 

This isn't quite as good as native HDR images technically, however users who consume these images still get the "HDR" benefit. On the other hand, people with a traditional SDR display don't have to worry about the complicated tone mapping mentioned previously, They will still receive the SDR picture as the original artist intended it. Whether this be Digital artist or a photographer, Both SDR and HDR users can enjoy a first class experience. 

So after all this, does JXL support gain maps? Well no. However readers may be pleased to hear that JXL allows encoding arbitrary information as their own channels. This means JXL can natively support encoding gain map images, and while there is no spec for this the potential for this use case is there.

## I have a need for speed that can't be sated, but oh boy does JXL get close.

Decode speed of lossy JXL images is actually really fast. Ok, yeah sure, even a potato can decode as single image. but what about when you have 30 images on screen at once. "Who needs that?" one may ask. and the answer is Galleries! There are hundreds of galleries on the web, in our phones, and on our PCs. Ok, well JXL can be useful here for sure. But when I say I have a need for speed, **I mean real speed**.

Something I have been testing (although because of FFmpeg updates, this is now broken, woes of unofficial patches,) Is using JXL is a mezzanine format. A very quick TLDR on what a mezzanine format is, it's a video format comprised of a sequence of still photos designed to make video editing, well not suck. One needs only still images because that makes frame perfect seeking instant, and fast enough decode speed to make scrubbing smooth enough. 

* 30fps? Beginner driver or elderly driver? can't tell.
* 60fps? Maybe you can enter turtle races. (Lossless jxl can be found here. Magically faster then PNG sequences for me somehow... I think libjxl devs sold their souls to the devil.)
* 90fps+ Ah, here you are JXL!

Yeah, that's pretty fast, "But Quack! Cineform is much faster then that! And ProRes is even faster!!!" Okay yeah that is true. I lost on this battle, but they are also about 2x the size for similar quality. If you run out of SSD space for your mezzanines, it has happened to me twice, and I swear it will not again. (I said this the first time too.) You will learn the true meaning of slow. 

I'm not bull shitting either, this was taken from march of this year (2023) 98FPS with MPV under Wayland using Vulkan + mailbox. I was getting from 90-120FPS when I was trying this! ![marine-unison](https://cdn.discordapp.com/attachments/549650852839686153/1087020829969227926/image.png)

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
![Ina is a comfy streamer, A shame you can't see this image.](https://files.catbox.moe/5obp2y.jxl)  

For this next image, This is "The Woag". It is an 8k image lossless encode using `cjxl -e 7 -d 0` with Jpeg reconstruction. This image might even slow down a desktop browser!

```  
root@archiso ~ # hyperfine --runs 5 'djxl thewoag.jxl --disable_output'
Benchmark 1: djxl thewoag.jxl --disable_output
  Time (mean ± σ):      8.871 s ±  0.017 s    [User: 15.441 s, System: 1.623 s]
  Range (min … max):    8.841 s …  8.883 s    5 runs
  
hyperfine --runs 5 'djxl thewoag.jxl --disable_output'  78.02s user 8.58s system 190% cpu 45.428 total
```
![the woag is pretty big! This might be slowing down JXL enabled browsers though.](https://files.catbox.moe/fsmdyy.jxl)

And finally we have an image encoded with `cjxl -e 7 -d 1` from a PNG source.
```
root@archiso ~ # hyperfine --runs 5 'djxl mona.jxl --disable_output'
Benchmark 1: djxl mona.jxl --disable_output
  Time (mean ± σ):      5.944 s ±  0.007 s    [User: 10.453 s, System: 0.925 s]
  Range (min … max):    5.934 s …  5.954 s    5 runs

hyperfine --runs 5 'djxl mona.jxl --disable_output'  53.07s user 5.03s system 188% cpu 30.790 total
```

![I may have a small addiction to genshin, The game is great and so is the art you are missing!](https://files.catbox.moe/6o1t4r.jxl)

As you can see, Even on a really old 1GB dual core atom, JXL is surprisingly fast. I wouldn't recommend browsing a JXL enabled page. But I also wouldn't reccomend using one of these machines in the first place. It is also worth noting that this is an Arch linux Live boot, not a full install.

Ina: https://files.catbox.moe/5obp2y.jxl

Thewoag: https://files.catbox.moe/fsmdyy.jxl

Mona: https://files.catbox.moe/6o1t4r.jxl

## OK, JXL is kinda cool. I would use it but man, C++ sucks. 

I hear you loud and clear. If it was any louder I might go deaf. I know the pain, and I hate it just as much. However [have no fear, Rust is here!](https://github.com/tirr-c/jxl-oxide) Well for decoding at least. But still! jxl-oxide is a third party, native rust decoder, and it is pretty fast. The important part of jxl-oxide however, Is not the MIT license, It's not that rust is the best programming language ever. It's this, 

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

That's right, with a simple cargo build command line, we can make a 3.4M binary, with proper static compilation so it will run on almost any Linux system, and when we use UPX to compress it, it's a mere 966K. No need to muck about dependencies. No need for weird cross compile shenanigans and these sizes are without rebuilding std.

Thanks to being rust, this also means you can easily use jxl-oxide crate in your projects, one could also probably build jxl-oxide as a library and use it in other languages too. Though I'm not sure if anyone has done it. There is a [wasm demo](https://gitlab.com/nitroxis/jxl-wasm) too for people who would want to use it targeting that.

Of course, one should keep in mind that jxl-dec binary does have some limitations, jxl-dec It only supports PNG and npy output. You don't seem to be able to specify output colorspace either at the moment. 

But how does it preform on that crappy laptop from before? Well here are the results from the mona image!

```
this should be something here
```

## A great colour space, Easy decoder, Amazing flexibility and speed to boot! What am I loosing here chief?

Well, Support, JXL is still a new format, at the time of writing, you wont find it any any android gallery app that I know off, only a couple apps I know of actually support it. 

You also wont find support in very many web browsers either with chrome removing support, and Firefox being Firefox and ignoring the issue outright. However thanks to apple of all people, at least Webkit supports JXL. Who would have thought that the people fighting against the chrome monopoly, was not Firefox but apple of all people... who woulda thought.

Linux support is not a thing. People have a misconception about how Linux handles these things. Linux relies on the application themselves to support nearly everything. There is no generic platform support for these. There are just... a lot of platforms to choose from some more generic then others. 

Thankfully however, independent app ecosystems are generally pretty good, both QT and GTK support JXL, dedicated tooling like imagemagick and FFmpeg support it too. So you shouldn't have too many issues here.

As alluded to above, Apple is doing great here. They are offering platform support for JXL now so you should have zero issues here soon enough as long as you are up to date. 

As for windows, there is a plugin to add it to windows imaging component. It works ok, it's pretty limited but its... fine. It of course is not official support. This can and will cause issues with adoption at this point.

You can help push adoption by recommending it to people who can use it, and reporting missing features as bugs, and for support as feature requests. Websites are already serving JXL too, so you can report those websites as broken.