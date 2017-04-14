---
layout: post
title: From Unity to Gnome
tags: ubuntu, unity, gnome, desktop manager, linux
---

Gnome is my new home. I did not plan to move. Following is the thought and click process that took me there. You can just skip it to the next part on [fixing it up](#fixing-up-gnome). 

* I was looking for a way to setup a new Firefox profile.
* And a way to have windows with different profiles open at the same time.
* Which lead me to Firefox addons website.
* On its front page, I saw an interesting featured addon.
* I clicked and navigated to its page.
* It was an extension for integrating with Gnome shell and allow extension installation.
* I did not understand what exactly it was.
* I followed a link to the Gnome extensions website.
* Saw an interesting extension: Random Wallpaper.
* It sits nicely in the Gnome panel and allows you to change wallpaper from there.
* Interesting, I have built a similar utility which does that from command line.
* May be, I should add such a shell extension for it, it would be nice.
* But, how do I program a Gnome shell extension?
* Googled it.
* Hit a few tutorials.
* Found a short, helpful one: <http://smasue.github.io/gnome-shell-tw>
* Hey, but, I think the default desktop manager for Ubuntu is Unity.
* Does it support shell extensions?
* It turns out, it does not.
* So, i'll have to install Gnome window manager to have shell extensions.
* Should I really  do it?
* Let's see some comparison.
* This article here: <https://www.lifewire.com/ubuntu-unity-vs-gnome-2201173> compared and contrasted.
* And preferred Gnome for being a little fast.
* That was good enough to go on by.
* Let's hit the Gnome 3 website and check it out.
* What I saw on its front page blew my mind.
* Nice features and awesome looks.
* I wanted it.
* I hit `> sudo apt-get install gnome-shell`
* Some 27 MB of download with apprx. 50 MB of estimated install size.
* Recently, coming back from windows world, it was nothing.
* It installed.
* I logged out.
* And logged back in after selecting Gnome at the login prompt.
* That's it.
* I had moved from Unity to Gnome.
 
-

All this time, I never really thought about it much. I knew the name of the window manager on Ubuntu was Unity. But, I had also seen and used a lot of g* applications on Unity to assume it was somehow Gnome. It is not. Ubuntu switched its default window manager from Gnome to Unity with Ubuntu 11. Unity is developed by Ubuntu, whereas, Gnome is developed by the Gnome community.

-

<a name="fixing-up-gnome"></a>
### Fixing up Gnome

After logging in, the first thing I noticed was how it did not look anything like the screenshots I saw on its website. Following are the steps through which I fixed that.

1. Install Gnome tweak tool.

2. Tweak the UI settings.

3. Install a good theme. I recommend the Paper theme. It has a nice flat scheme, an icon set and a cursor set. (<https://snwh.org/paper>)

4. Install gnome-shell-extensions (via apt-get)

5. Logoff and log back in for the extensions to show up in the Gnome tweak tool.

6. Feel free to toggle extensions, there are many.

7. Install the Firefox Gnome shell integration addon mentioned before (<https://addons.mozilla.org/en-US/firefox/addon/gnome-shell-integration/>). This will let you install Gnome extensions from the Gnome extensions website (<https://extensions.gnome.org/>) directly. It is unbelievable that there are no direct download links on this website. So, you cannot just download the extension archive and extract it to the extensions directory yourself. BTW, the extensions directory is: ~/.local/share/gnome-shell/extensions. That's where you go if you want to tweak your extension.

8. Install the extension: Activities configurator if you are not happy with the default activities look. I changed the name "Activities" to "gnome". You can also add an icon there via this extension and tweak more things.

9. Enable the User themes extension if you want to use your own themes.

10. Next, if you don't like the clock being right in the middle of the top panel, you can move it using the extension: "Frippery move clock". It has no configurable settings. It just moves the clock to right when enabled. Unfortunately, it does not move it all the way to the right like that in Unity. It places it to the left of the "network-volume-battery" status set.

11. The top panel looked bad. It was thicker than it needed to be and its font was large and bold. That was probably due to the css of my theme: Paper. I opened its css (gnome-shell.css) and edited away the things I did not like. Thanks to the guide here: <http://rlog.rgtti.com/2012/01/29/how-to-modify-a-gnome-shell-theme/>, I knew what settings to edit / add.

12. Next, Gnome does not merge the title bar of a window into the top panel when the window is maximized. Coming from Unity, you'd expect that. There is an extension for that: "Pixel saver". Install it and you will save some screen real estate every time you maximize a window.

13. Gnome for some reason starts all windows maximized. You can hit the resize button to bring it back to the last used size, but, would you want to do that every time? Thankfully, I found a solution:
`> gsettings set org.gnome.mutter auto-maximize false`

14. It'd be good to have the numbers feature on the application launcher, like that in Unity. As expected, there is an extension for it: "Appkeys". But, remember you have to hit the number before you release the "app key" (windows key for a typical windows keyboard).


With that I had the Gnome desktop working with the desired look and features. Of course, I still lacks a few things that Unity has. It does not show the active application menu in the top panel like Unity does. I could not find any extensions for it either.

But, overall, a positive upgrade.

The client side decorations (which give the applications a nice flat title bar and the new gtk header bar) is on by default and applies to all applications. On Unity, I had to enable it by editing the profile of each application. And it still would not take effect on many applications.

The feature that pushed me to Gnome, the shell extensions are awesome. There are numerous extensions available. And the possibility of programming one is now closer.

It looks great.

If I were Sheldon Cooper, I would say, "yes, this is definitely going to be my spot"!

-
