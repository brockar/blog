+++
author = "Hugo Authors"
title = "Placeholder Text"
date = '2019-03-09'
description = "Lorem Ipsum Dolor Si Amet"
categories = [
    "Test",
    "Test with whitespaces"
]
tags = [
    "markdown",
    "text",
    "tag with whitespaces"
]
image = "img.jpg"
+++

---

## YouTube Privacy Enhanced Shortcode

{{< youtube ZJthWmvUzzc >}}

<br>

---

## Twitter Simple Shortcode

{{< twitter_simple user="DesignReviewed" id="1085870671291310081" >}}

<br>

---

## Gist Shortcode

{{< gist spf13 7896402 >}}

## Gitlab Snippets Shortcode

{{< gitlab 2349278 >}}

## Quote Shortcode

Stack adds a `quote` shortcode.  For example:

{{< quote author="A famous person" source="The book they wrote" url="https://en.wikipedia.org/wiki/Book">}}
Lorem ipsum dolor sit amet, consectetur adipiscing elit, ...
{{< /quote >}}

{{< quote source="Anonymous book" url="https://en.wikipedia.org/wiki/Book">}}
Lorem ipsum dolor sit amet, consectetur adipiscing elit, ...
{{< /quote >}}

---

<p>
<span class="nowrap"><span class="emojify">🙈</span> <code>:see_no_evil:</code></span>

<span class="nowrap"><span class="emojify">🙉</span> <code>:hear_no_evil:</code></span>  

<span class="nowrap"><span class="emojify">🙊</span> <code>:speak_no_evil:</code></span>
</p>


The [Emoji cheat sheet](http://www.emoji-cheat-sheet.com/) is a useful reference for emoji shorthand codes.

**N.B.** The above steps enable Unicode Standard emoji characters and sequences in Hugo, however the rendering of these glyphs depends on the browser and the platform. To style the emoji you can either use a third party emoji font or a font stack; e.g.

{{< highlight html >}}
.emoji {
  font-family: Apple Color Emoji, Segoe UI Emoji, NotoColorEmoji, Segoe UI Symbol, Android Emoji, EmojiSymbols;
}
{{< /highlight >}}

{{< css.inline >}}
<style>
.emojify {
	font-family: Apple Color Emoji, Segoe UI Emoji, NotoColorEmoji, Segoe UI Symbol, Android Emoji, EmojiSymbols;
	font-size: 2rem;
	vertical-align: middle;
}
@media screen and (max-width:650px) {
  .nowrap {
    display: block;
    margin: 25px 0;
  }
}
</style>
{{< /css.inline >}}

---
# Links
This page's frontmatter:

```yaml
title: Links
Author: brockar
links:
  - title: GitHub
    description: GitHub is the world's largest software development platform.
    website: https://github.com/brockar
    image: https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png
```

or 

```TOML
[[links]]
title = "Ubuntu setup"
website = "https://ubuntu.com/tutorials/install-and-configure-samba#4-setting-up-user-accounts-and-connecting-to-share"
image = "https://repository-images.githubusercontent.com/335476519/61ef634e-0b5f-4d27-9fb6-c64d526c595c"

[[links]]
title = "Odisea Geek"
website = "https://odiseageek.es/posts/montar-unidades-smb-o-cifs-en-linux-con-mount-o-fstab/"
image = "https://avatars.githubusercontent.com/u/60941836?v=4"
```