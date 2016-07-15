---
layout: post
title: Upgrading / Installing Python Pillow On Windows
tags: windows, python, pillow, imaging, install, wallp
---

I was minding my own business, upgrading my own utility [wallp](https://github.com/amol9/wallp) on Windows (10). Having done this many times before, I expected it to finish in a few minutes.
Since, I issued an upgrade command, all the dependencies were getting upgraded too.

 {% highlight powershell %}
 > pip install --upgrade wallp
 {% endhighlight %}

During the process, it downloaded the huge Pillow source archive and tried to install it. It failed and all hell broke loose.

It tried to build the package from sources and it didn't work since, I don't have the development environment (Visual Studio) and the libraries setup for it.
The error said something about missing zlib and installing using '--disable-zlib'. I wanted zlib, since, my code handles png and jpeg images.

This was new, since, I had installed the previous version (3.2.0) of pillow using pip on the same machine and it did so without any hiccups.

Here are the things I tried to get it upgraded / installed:

### 1. Download the appropriate wheel

 {% highlight powershell %}
 > python -c 'import pip; print(pip.pep425tags.get_supported())'
 
 [('cp34', 'cp34m', 'win32'), ('cp34', 'none', 'win32'), ('py3', 'none', 'win32'), ('cp34', 'none', 'any'), ('cp3', 'none
 ', 'any'), ('py34', 'none', 'any'), ('py3', 'none', 'any'), ('py33', 'none', 'any'), ('py32', 'none', 'any'), ('py31', '
 none', 'any'), ('py30', 'none', 'any')]
 {% endhighlight %}

 So, I downloaded Pillow-3.3.0-cp34-cp34m-win32.whl and tried to install it.

 {% highlight powershell %}
 > pip install .\Pillow-3.3.0-cp34-cp34m-win32.whl
 Pillow-3.3.0-cp34-cp34m-win32.whl is not a supported wheel on this platform.
 {% endhighlight %}

 As you can see, all the tags for the wheel are matching, still, it did not install.


### 2. Skip the installation

 Why not just skip the upgrade for pillow and finish the installation? So, I just uninstalled wallp and reinstalled it. It went fine. Except, that, the first attempt at install had already uninstalled previous version of Pillow. So, now, wallp won't run.


### 3. Rename directory 'pil' to 'PIL'

 While searching the Internet, I came across this suggestion. So, I checked the site-packages directory. The directory was named 'pil'. So, I renamed it to 'PIL'.

 The only difference it made was that it changed the error from:

 ImportError: No module named 'PIL'
 {: style="color:red;"}

 to:

 ImportError: cannot import name 'Image'
 {: style="color:red;"}


### 4. Try the executable installer

 Downloaded Pillow-3.3.0.win32-py3.4.exe and installed it.

 Still the same error: 

 ImportError: cannot import name 'Image'
 {: style="color:red;"}
 

### 5. Remove existing installation by hand and reinstall

 I just renamed the directories:

 - pillow-3.2.0.dist-info
 - Pillow-3.3.0-py3.4.egg-info

 and reinstalled using executable installer.

 Still the same error:

 ImportError: cannot import name 'Image'
 {: style="color:red;"}


### 6. Remove existing 'PIL' directory

 I noticed that, the files in the PIL directory weren't touched despite the reinstall.
 So, I renamed that directory as well.
 And, reinstalled using executable installer.

 And, it worked !!

 A test statement in python repl executed without errors.

 {% highlight python %}
 >>> from PIL import Image
 {% endhighlight %}

 A check using `pip list` showed Pillow with version 3.3.0.

-

I had already spent an hour on this simple thing by try #4 and was pulling my hair out. It was dinner time. I had to give up.
But, next day, I figured out the solution by tring #5 and #6 which happened to hit me in the shower in the meantime.

If the otherwise smooth slithering beasts that are python, pypi, pip and pillow would have just picked the right wheel and installed it, it would have saved me a ton of pain.

