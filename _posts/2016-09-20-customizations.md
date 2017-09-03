---
layout: post
title: "Customizations"
date: 2016-09-20 16:25:06
description: Few customizations available out of the box!
share: true
tags:
 - customizations
 - jekyll
---

# Accent color

Accent color is color for some important elements, such as links, buttons, icons. Currently accent color is <span class="label" style="background-color:#3CA2A2; color:#444444">#3CA2A2</span>. This theme has some more predefined colors available in **theme.scss**:

>theme.scss
{:.filename}
{% highlight sass %}
 // Several accent colors, choose one or create your own!
 $accent-color: #3CA2A2;    // original =)
 //$accent-color: #C38FD6;  // velvet
 //$accent-color: #8FD6B3;  // greenish
 //$accent-color: #35B4DE;  // bluish
 //$accent-color: #D2E354;  // yellowish
 // $accent-color: #52B54B;  // green
 
{% endhighlight %}

You can use one of them (just hover over the label to see accent color in action) or define your own!

<span class="label" style="background-color:#C38FD6; color:#444444">#C38FD6</span>, <span class="label" style="background-color:#8FD6B3; color:#444444">#8FD6B3</span>, <span class="label" style="background-color:#35B4DE; color:#444444">#35B4DE</span>, <span class="label" style="background-color:#D2E354; color:#444444">#D2E354</span>, <span class="label" style="background-color:#52B54B; color:#444444">#52B54B</span>. 
 
<script>
  $('.label').hover(function(){
    var color = $(this).text();
    [].forEach.call($('a'), function(item) {
      item.style.color = color
    })
  })
</script>

<style>
  .label{
    cursor: default;
    border-radius: 5px;
    padding: 5px 8px;
  }
</style>

# Other colors

As Jekyll comes with support of SASS I put colors in variables. Here are the ones which could be easily changed:

>theme.scss
{:.filename}
{% highlight scss %}
$font-color: #dddddd;
$background-color: #292929;
$post-panel-color: #444;
$footer-background-color: #292c2f;
$note-color: #87CEFA;
$warning-color: #ffff00;
{% endhighlight%}