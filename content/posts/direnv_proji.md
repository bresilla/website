+++ 
show = true
type = "post"
date = "2019-12-23"
title = "How to manage projexts, the *NIX way!"
slug = "direnv-nixshel-proji" 
tags = ["direnv","nix-shell","proji" ]
categories = ["DEV"]
series = ["DIRENV"]
part = "1"
styleaccent = "#fef460"
styleforeground = "#c4d0d1"
stylebackground = "#080a0f"
footercontent = "Things will not calm down, as a matter of fact they will just calm up - Teal'c (Stargate)"
width="60%"
+++


{{< imfra url="https://i.imgur.com/momvy3U.png" border="1" width="70" >}} Rofi menu: Listing all active projects {{< /imfra >}}

{{< texfra type="fill" >}}
<details class="dropdown">
  <summary>Finished skills</summary>
{{< progressbar 70 ffffff >}} VIM {{< /progressbar >}}
{{< progressbar 90 ffffff >}} Tailing VM (bspwm) {{< /progressbar >}}
{{< progressbar 85 ffffff >}} LATEX {{< /progressbar >}}
{{< progressbar 80 ffffff >}} Containers {{< /progressbar >}}
{{< progressbar 60 ffffff >}} NixOS & NixShell {{< /progressbar >}}
{{< progressbar 70 ffffff >}} ROS2.0 {{< /progressbar >}}
{{< progressbar 30 ffffff >}} Unreal Engine {{< /progressbar >}}
{{< progressbar 40 ffffff >}} Home Assistant {{< /progressbar >}}
</details>
{{< /texfra >}}

## Introduction

This tutorial will show you how to create a simple theme in Hugo. I assume that you are familiar with HTML, the bash command line, and that you are comfortable using Markdown to format content. I'll explain how Hugo uses templates and how you can organize your templates to create a theme. I won't cover using CSS to style your theme.

We'll start with creating a new site with a very basic template. Then we'll add in a few pages and posts. With small variations on that, you will be able to create many different types of web sites.

In this tutorial, commands that you enter will start with the "$" prompt. The output will follow. Lines that start with "#" are comments that I've added to explain a point. When I show updates to a file, the ":wq" on the last line means to save the file.

Here's an example:

{{< highlight go >}}
package main

import "fmt"

func main() {
	fmt.Println("Hello, World!")
}
{{< / highlight >}}


## Some Definitions

There are a few concepts that you need to understand before creating a theme.

#### Skins

Skins are the files responsible for the look and feel of your site. It’s the CSS that controls colors and fonts, it’s the Javascript that determines actions and reactions. It’s also the rules that Hugo uses to transform your content into the HTML that the site will serve to visitors.

You have two ways to create a skin. The simplest way is to create it in the ```layouts/``` directory. If you do, then you don’t have to worry about configuring Hugo to recognize it. The first place that Hugo will look for rules and files is in the ```layouts/``` directory so it will always find the skin.

Your second choice is to create it in a sub-directory of the ```themes/``` directory. If you do, then you must always tell Hugo where to search for the skin. It’s extra work, though, so why bother with it?

The difference between creating a skin in ```layouts/``` and creating it in ```themes/``` is very subtle. A skin in ```layouts/``` can’t be customized without updating the templates and static files that it is built from. A skin created in ```themes/```, on the other hand, can be and that makes it easier for other people to use it.

The rest of this tutorial will call a skin created in the ```themes/``` directory a theme.

Note that you can use this tutorial to create a skin in the ```layouts/``` directory if you wish to. The main difference will be that you won’t need to update the site’s configuration file to use a theme.
