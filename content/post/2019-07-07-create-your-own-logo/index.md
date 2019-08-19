---
title: How to create your own logo and apply to your own website
author: 
- Yuan Du
date: '2019-07-07'
slug: create-your-own-logo
categories:
  - R
tags:
  - Blogdown
---

I have no background of editing html and css. It took me a while to figure out how to modify the [Hugo Lithium theme](https://github.com/yihui/hugo-lithium) for my own blog. so I would like to share it with you, by no means that this is the only way to do it.

**Step 1:**
Generate your own favicon ico by using free [favicon generator](https://favicon.io/). I generated it by using my inital of my first name. 

**Step 2:**
Save the download package and save the `favicon.ico` logo in the path `\thmemes\hugo-lithium\static\` (Hugo will copy it to root directory).and copy the small logo named `apple-touch-icon.png` under the path `\thmemes\hugo-lithium\static\images`

**Step 3:**
change the url `logo.png` name in the file `config.toml` under 

`[params.logo] url = "apple-touch-icon.png" `
