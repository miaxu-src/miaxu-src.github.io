---
layout: post
---

Jekyll supports source code syntax highlighting. However, I found an issue for the theme "minima" that comes with `jekyll new my-website`: If I want to showe line numbers for my code block, too much white space is rendered between a line number and my source code. To avoid the hassel of dealing with the default theme, here is what I have done.

1. I disabled the theme "minima" in my `_config.yml` and downloaded a stylesheet `native.css` from <a href="https://jwarby.github.io/jekyll-pygments-themes/languages/ruby.html">this website</a>. I changed the file extention to `.scss` (VERY important!) and saved it into the `_sass` folder of my website.

2. In the file `assets/css/styles.scss`, added a line `@import "native"` to source the native.scss into my stylesheet.

After the above steps, it gave me a nice syntax highlighting with line numbers displayed in my code block.

To make the line numbers look even better, I also added following lines into my `_sass/main.scss`, which adds padding and border for my line numbers:

{% highlight scss linenos %}
.lineno { 
  color: #ccc; 
  display: inline-block; 
  padding: 0 5px; 
  border-right:1px solid #ccc;
}

.highlight table td.gutter {
    width: 10px;
    padding-right: 1em;
    color: #bdc1c4;
}
{% endhighlight %}

