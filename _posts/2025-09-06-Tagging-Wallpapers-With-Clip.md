---
layout: post
title: Automatically Tagging Wallpapers with CLIP
categories: [Miscellaneous]
---

## Automatically Tagging Wallpapers with CLIP

Lately, I’ve been organising the images I use as [wallpapers](https://github.com/sdysch/dotfiles/tree/2fe1b6d86f84a23c95ae6bff3fcf979e128efb0e/wallpapers) on my computer, most of which I source from [r/wallpapers](http://reddit.com/r/wallpapers).  

A recurring problem was that many filenames were completely non-descriptive things like `2r3tpqp15vt41.jpg`, making it tricky to sort or browse through them.  

To attempt at sorting them, I used [CLIP](https://huggingface.co/sentence-transformers/clip-ViT-B-32), an image-text model, to automatically generate descriptive tags and append them to filenames.
Not perfect, but gets the job done.

The script is simple: provide a directory of images, and the script adds the top `k` predicted tags (default 1) to each filename. You can find the code [here](https://github.com/sdysch/wallpaper_tagger).  

For example, with a folder `./figs` containing several images, you can run:

```
python tag_wallpapers.py ./figs/ --top-k 1
```

After running the script, an image of Budapest that was previously named `Budapest.jpg` is now renamed to `Budapest_city.jpg`:  

![](/images/250906/Budapest_city.jpg)

Simple, but maybe useful.
