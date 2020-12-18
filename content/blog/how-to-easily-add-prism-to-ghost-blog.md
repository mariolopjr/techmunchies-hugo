---
title: How to Easily Add Prism to Ghost Blog
date: '2018-01-06T09:56:34-05:00'
description: >-
  When it comes to presenting code on a blog, nothing beats syntax highlighting.
  And if you've recently converted to Ghost, it comes with no syntax
  highlighting out of the box, which is a pain.
---
When it comes to presenting code on a blog, nothing beats syntax highlighting. And if you've recently converted to Ghost, it comes with no syntax highlighting out of the box, which is a pain. Luckily, Ghost has the built-in ability to add code in the header or footer of the blog (so your changes do not get wiped out on the next update to Ghost) and I have the necessary code below to get it all working!

The code below will load the Prism script from the Cloudflare CDN, which means it may already be cached on the user's system (and making your site faster!). Additionally, it includes a couple of plugins and my custom JavaScript helpers[^1] that make your life easier, detailed below. I also included a little primer on how to display code in your blog posts. While some tutorials will have you modifying your theme, I like the flexibility of using Ghost's Code injection to keep my theme free from custom edits. First, install time!

### Installation

1. Navigate to your blog's Admin side.
2. On the left hand navigation, click on <svg version="1" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" height="15px" width="15px"><path d="M9.354 3.147a.5.5 0 0 0-.707 0l-8.5 8.5a.5.5 0 0 0 0 .707l8.5 8.5a.498.498 0 0 0 .707 0 .5.5 0 0 0 0-.707L1.207 12l8.147-8.146a.5.5 0 0 0 0-.707zm14.5 8.499l-8.5-8.5a.5.5 0 0 0-.707.707L22.793 12l-8.146 8.146a.5.5 0 0 0 .707.708l8.5-8.5a.502.502 0 0 0 0-.708z"></path></svg> Code injection.
3. Copy the following code into the Blog Header box under any existing code.

```html
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/prism/1.9.0/themes/prism-okaidia.min.css" integrity="sha256-e1bqZBSPbpH18ISDr1aR1mwb4PWTG/3QFxE51/22n14=" crossorigin="anonymous" />
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/prism/1.9.0/plugins/line-numbers/prism-line-numbers.min.css" integrity="sha256-SA3HrKLu4tldegZQVdUx8Cwwo2hQFIu7iW6UOo6OdS4=" crossorigin="anonymous" />
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/prism/1.9.0/plugins/command-line/prism-command-line.min.css" integrity="sha256-QViEGvwX/42OrD49huSb2mGZtUQwVKNA5Ux58tMWLOs=" crossorigin="anonymous" />
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/prism/1.9.0/plugins/toolbar/prism-toolbar.min.css" integrity="sha256-oqIQBM3BkdYsUyIdtwZql/C/XN5mGcz3LERZCJU/C0I=" crossorigin="anonymous" />
```

<br>

4. Copy the following code into the Blog Footer box under any existing code.

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/clipboard.js/1.7.1/clipboard.min.js" integrity="sha256-Daf8GuI2eLKHJlOWLRR/zRy9Clqcj4TUSumbxYH9kGI=" crossorigin="anonymous"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/prism/1.9.0/prism.min.js" integrity="sha256-hK55yYjZp3vXqUhowkWYVaoavcWSYRR6fE8sgHqyt5A=" crossorigin="anonymous"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/prism/1.9.0/plugins/autoloader/prism-autoloader.min.js" integrity="sha256-/eFVLWtwHKExKyYUwicIThi9orKW0nxCqtU53BfLFpg=" crossorigin="anonymous"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/prism/1.9.0/plugins/line-numbers/prism-line-numbers.min.js" integrity="sha256-JfF9MVfGdRUxzT4pecjOZq6B+F5EylLQLwcQNg+6+Qk=" crossorigin="anonymous"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/prism/1.9.0/plugins/toolbar/prism-toolbar.min.js" integrity="sha256-QQEEvcdk8r1EU6emWNFc2AFPqgHTE5KBZ4jHkJX65i0=" crossorigin="anonymous"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/prism/1.9.0/plugins/copy-to-clipboard/prism-copy-to-clipboard.min.js" integrity="sha256-RMU7YkOvO7IV1StJVYbICQZ5SrD3s6UxpH0ZKiMexMg=" crossorigin="anonymous"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/prism/1.9.0/plugins/command-line/prism-command-line.min.js" integrity="sha256-jk1BXEBRzVfaP1/NTOMdME2eSWsEyoWlJsOLVi2deWs=" crossorigin="anonymous"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/prism/1.9.0/plugins/normalize-whitespace/prism-normalize-whitespace.min.js" integrity="sha256-31Ef0tfMJHz4mcBYno6PKR2Iz5qPHPJLxIzej2AGl6c=" crossorigin="anonymous"></script>
<script>Prism.plugins.autoloader.languages_path='https://cdnjs.cloudflare.com/ajax/libs/prism/1.9.0/components/',Prism.plugins.autoloader.ignored_language='none',Prism.plugins.NormalizeWhitespace.setDefaults({'remove-trailing':!0,'remove-indent':!0,'left-trim':!0,'right-trim':!0});var codes=$('code');codes.addClass('line-numbers'),$(document).ready(()=>{codes.each((a,b)=>{var c=$(b),d=c.hasClass('language-nonum');const e=/PS[a-zA-z]:\\/g;if(d){c.children('span.line-numbers-rows').remove(),c.removeClass('language-nonum');var f=c.attr('class'),g=f.substring(1,f.length).split('-');f='language-'+g[0],c.attr('class',f);var h=2<=g.length?'l'===g[1]?'localhost':g[1]:'none';'none'!==h&&(null===h.match(e)?c.parent().attr({'data-host':h}):c.parent().attr({'data-prompt':'PS '+h.slice(2)}),f+=' command-line');var j=3<=g.length?'ps'===g[2]?'none':g[2]:'none';'none'!==j&&c.parent().attr({'data-user':j});var k='';if(4<g.length){k=g[3];for(var a of g.slice(4))k+='-'+a}else 4===g.length&&(k=g[3]);4<=g.length&&c.parent().attr({'data-output':k}),c.parent().attr('class',f),Prism.highlightElement(c[0])}})});</script>
```

<br>

5. Click Save in the top right corner and you're done! Now all of your fenced code blocks will automatically look like the ones on this blog, awesome! :)

### Usage

Since my Prism installation is custom, and I wanted to not have to custom code ```<pre><code></code></pre>``` whenever I wanted to use a custom feature in Prism, I added JavaScript helpers to allow me to do some cool Markdown stuff with my code blocks.

To mark a section of text as code, Ghost (and most Markdown parsers) use three backticks [```](https://help.ghost.org/hc/en-us/articles/224410728-Markdown-Guide#writing-code). Use the three backticks to _fence_ in your code. Additionally, you can specify the syntax highlighter to highlight. Here's the general syntax:

###### Code
````md
```js
var materials = [
  'Hydrogen',
  'Helium',
  'Lithium',
  'Beryllium'
];

console.log(materials.map(material => material.length));
```
````

<br>

###### Actual
```js
var materials = [
  'Hydrogen',
  'Helium',
  'Lithium',
  'Beryllium'
];

console.log(materials.map(material => material.length));
```

<br>

Notice after the beginning three backticks, there's ```js```? That's the name of the language (it is short for ```javascript```). There are many supported [languages](http://prismjs.com/#languages-list). All of the languages are supported by my code above, I use the autoloader plugin to download the relevant language files on the fly.

However, what if you'd like to show some console output? Utilizing my awesome JavaScript helpers, you can do exactly that using just Markdown! :)

If you need to show general bash commands ran as root, you would do the following[^2]:

###### Code
````md
```nonum-bash-l-root
cd /usr/local/etc
cp php.ini php.ini.bak
vi php.ini
```
````

<br>

###### Actual
```bash
cd /usr/local/etc
cp php.ini php.ini.bak
vi php.ini
```

<br>

Let's break down the "language" I used. Notice how I used ```` ```nonum-bash-l-root```` instead of ```` ```bash````.

```nonum```: disables line numbers for the code block
```bash```: sets the language to bash
```l```: short for ```localhost```.
&ensp;&ensp; This is the hostname, only supports letters and numbers
```root```: sets the username for the user

Let's take a look at another example, with a standard user and some displayed output.

###### Code
````md
```nonum-bash-l-chris-2,4-8
pwd
/usr/home/chris/bin
ls -la
total 2
drwxr-xr-x   2 chris  chris     11 Jan 10 16:48 .
drwxr--r-x  45 chris  chris     92 Feb 14 11:10 ..
-rwxr-xr-x   1 chris  chris    444 Aug 25  2013 backup
-rwxr-xr-x   1 chris  chris    642 Jan 17 14:42 deploy
```
````

<br>

###### Actual
```bash
pwd
/usr/home/chris/bin
ls -la
total 2
drwxr-xr-x   2 chris  chris     11 Jan 10 16:48 .
drwxr--r-x  45 chris  chris     92 Feb 14 11:10 ..
-rwxr-xr-x   1 chris  chris    444 Aug 25  2013 backup
-rwxr-xr-x   1 chris  chris    642 Jan 17 14:42 deploy
```

<br>

This utilizes line numbers to display output as it would in a console application. Make sure there is no space when setting the line numbers.

```nonum```: disables line numbers for the code block
```bash```: sets the language to bash
```l```: short for ```localhost```.
```chris```: sets the username for the user
```2,4-8```: sets lines 2 and 4-8 as output.
&ensp;&ensp; Can be single numbers, ```2``` or a range ```4-9``` or both (like this example) ```2,4-8```.
&ensp;&ensp; Separate different output lines/ranges with commas.

Now, what's one application where whitespace _really_ matters? Ever use PowerShell and format its output as a table? Makes the output look great, however, correctly displaying the syntax and output in your Ghost blog has never been easier. Let's take a look:

###### Code
````md
```nonum-powershell-PSC:\Users\Chris>-ps-2-19
dir


    Directory: C:\Users\Chris


Mode                LastWriteTime     Length Name
----                -------------     ------ ----
d-r--        10/14/2015   5:06 PM            Contacts
d-r--        12/12/2015   1:47 PM            Desktop
d-r--         11/4/2015   7:59 PM            Documents
d-r--        10/14/2015   5:06 PM            Downloads
d-r--        10/14/2015   5:06 PM            Favorites
d-r--        10/14/2015   5:06 PM            Links
d-r--        10/14/2015   5:06 PM            Music
d-r--        10/14/2015   5:06 PM            Pictures
d-r--        10/14/2015   5:06 PM            Saved Games
d-r--        10/14/2015   5:06 PM            Searches
d-r--        10/14/2015   5:06 PM            Videos
```
````

<br>

###### Actual
```powershell
dir


    Directory: C:\Users\Chris


Mode                LastWriteTime     Length Name
----                -------------     ------ ----
d-r--        10/14/2015   5:06 PM            Contacts
d-r--        12/12/2015   1:47 PM            Desktop
d-r--         11/4/2015   7:59 PM            Documents
d-r--        10/14/2015   5:06 PM            Downloads
d-r--        10/14/2015   5:06 PM            Favorites
d-r--        10/14/2015   5:06 PM            Links
d-r--        10/14/2015   5:06 PM            Music
d-r--        10/14/2015   5:06 PM            Pictures
d-r--        10/14/2015   5:06 PM            Saved Games
d-r--        10/14/2015   5:06 PM            Searches
d-r--        10/14/2015   5:06 PM            Videos
```

<br>

```nonum```: disables line numbers for the code block
```powershell```: sets the language to powershell
```PSC:\Users\Chris>```: sets powershell prompt.
&ensp;&ensp; No spaces! Make sure the space between PS and your current path is removed.
```ps```: powershell doesn't show current user, so set as ```ps``` to skip.
```2-19```: sets lines 2-19 as output.

You may use nonum to not display line numbers for a code block. This may be useful for some reason?


###### Code
````md
```nonum-js
var materials = [
  'Hydrogen',
  'Helium',
  'Lithium',
  'Beryllium'
];

console.log(materials.map(material => material.length));
```
````

<br>

###### Actual
```js
var materials = [
  'Hydrogen',
  'Helium',
  'Lithium',
  'Beryllium'
];

console.log(materials.map(material => material.length));
```

<br>

As always, if there are any questions, please send me an email and I will try to answer them!


#### Aside
If you were wondering how I was able to show the code block syntax within a code block, then you will not be disappointed. Essentially, if you _nest_ the backticks, i.e. the outer backticks have an additional backtick than the nested one, then it will nest the code. This can (presumably) go on infinitely.

###### Code
`````md
````none
```nonum-bash-l-root
cd /usr/local/etc
cp php.ini php.ini.bak
vi php.ini
```
````
`````

<br>

#### Plugins
I've listed the plugins I am using with Prism, click on the plugin name to learn more.

[Prism](http://prismjs.com)
[Autoloader](http://prismjs.com/plugins/autoloader)
[Command Line](http://prismjs.com/plugins/command-line)
[Copy to Clipboard](http://prismjs.com/plugins/copy-to-clipboard)
[Line Numbers](http://prismjs.com/plugins/line-numbers)
[Normalize Whitespace](http://prismjs.com/plugins/normalize-whitespace)
[Toolbar](http://prismjs.com/plugins/toolbar)


[^1]: My helper functions may be viewed at the following [gist](https://gist.github.com/mariolopjr/926875d4e7b6453378d2be5091cdfdbc)
[^2]: All examples shamelessly ripped from command line's original examples [here](http://prismjs.com/plugins/command-line/)
