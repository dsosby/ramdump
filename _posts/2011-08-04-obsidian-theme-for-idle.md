---
title: Obsidian Theme for IDLE
date: 2011-08-04
layout: post
categories:
 - Python
tags: []
---

I attempt to use dark backgrounds in all of my editors. I find it much <a href="http://www.thebestpageintheuniverse.net/c.cgi?u=faq" target="_blank">easier on my eyes</a> when I'm staring at code all day long or hacking on something early in the morning. I generally use something close to my favorite color scheme on Notepad++, Obsidian, which I  believe is based on Obsidian Coast in KDE. 

With that said, Python's IDLE application is harshly bright compared to my normal editors. After a quick Google search I found only a few themes available for IDLE, so I created IDLE highlighting settings that mimic my favorite color scheme. Here is the first draft for all to enjoy. You can also find the latest updates in <a href="https://gist.github.com/1122904" target="_blank">this</a> Gist repo. Installation is easy: just copy and paste these settings into your .idlerc\config-highlight.cfg file in your home directory (creating as necessary), then choose it in your Highlighting settings in IDLE by selecting "Use Custom Theme" and "Obsidian" from the dropdown.

<a href="http://ramdump.files.wordpress.com/2011/08/idle_obsidian.png"><img src="http://ramdump.files.wordpress.com/2011/08/idle_obsidian.png?w=265" alt="Screenshot of Obsidian for IDLE" title="IDLE_obsidian" width="265" height="300" class="aligncenter size-medium wp-image-67" /></a>

[code]
[Obsidian]
definition-foreground = #678CB1
error-foreground = #FF0000
string-background = #293134
keyword-foreground = #93C763
normal-foreground = #E0E2E4
comment-background = #293134
hit-foreground = #E0E2E4
builtin-background = #293134
stdout-foreground = #678CB1
cursor-foreground = #E0E2E4
break-background = #293134
comment-foreground = #66747B
hilite-background = #2F393C
hilite-foreground = #E0E2E4
definition-background = #293134
stderr-background = #293134
hit-background = #000000
console-foreground = #E0E2E4
normal-background = #293134
builtin-foreground = #E0E2E4
stdout-background = #293134
console-background = #293134
stderr-foreground = #FB0000
keyword-background = #293134
string-foreground = #EC7600
break-foreground = #E0E2E4
error-background = #293134
[/code]