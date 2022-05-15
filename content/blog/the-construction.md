---
title: "The construction of this blog!"
draft: false
---

## Introduction

Hi there, 

Finally I got myself a homepage! As someone who likes simplicity, a static website hosted on Github Pages seems like the perfect solution for this. 

This is a pretty simple blog, I want it to load fast on machine, and I hope it is readable enough!

What should I put on my blog post? So far I have: 

- My contact information.
- Categorized blogposts, 1 category only so far!
- One or two blogposts, including this one.

What to do next? 

- Think of stuff to write on this blog.
- Study more so I have stuffs to write.

## How is this blog constructed?

I use the Hugo framework, and modified the [Xmin](https://github.com/yihui/hugo-xmin) theme.

To modify any parts of the theme, you copy the files (for example the CSS files) and place them into the `static` folder in Hugo directory. By modifying the CSS, you can have text glow, dark theme, different text size.

Similarly, copy from the theme's `layout` folder to the root `layout` folder, 

Modify the `head_custom.html` so you have neaty stuffs like MathJax math rendering, and different fonts!

Finally, you can use Github Pages to host our files. Refer [this](https://gohugo.io/hosting-and-deployment/hosting-on-github/) to set up an automated build that rebuild your site for each commit.

Refer to the source code [here](https://github.com/grevyarlesp/homepage).