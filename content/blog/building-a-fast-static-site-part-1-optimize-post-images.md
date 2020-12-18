---
title: 'Building a Fast Static Site: Part 1 - Optimize Post Images'
date: '2016-08-01T09:51:19-04:00'
description: >-
  Building a fast static site (with Jekyll) can be daunting unless you know what
  you're doing. These​ tutorial series help you create a successful, fast site!
---
I've been wanting to automate my image optimization for this site. Normally, my workflow for uploading images for a blog post is as follows:

1. Download image to Desktop
2. Resize image to 1000px
3. Run image through ImageOptim to compress 70%
4. Upload to images directory

As you can see, quite a bit of steps. After some gulp wizardry (of which, I will introduce you to in a bit), my workflow is now as follows:

1. Upload to images directory

See how that is A LOT more desirable? At least, I think so!

Anyways, let's continue to how to set this up.

> Note: I assume (you know what that means!) that you have knowledge in gulp. If not, do not fear! I will upload a tutorial in the future detailing how to get up and running with gulp.

> Warning: I use gulp-load-plugins. This allows me to load any gulp plugin using $.pluginName (plugin name drops "gulp-", and becomes camelCase instead of dash-case). To accomplish this awesomeness, implement the following changes to your gulpfile.js:

1. Add the following to your package.json dependencies section  
   `"gulp-load-plugins": "1.2.4"`
2. Add the following to your plugin requires at the top of your gulpfile.js (Sets up gulp plugin loading using
   $.pluginName)  
   `var $ = require('gulp-load-plugins')();`

Make sure to have a gulpfile.js in your project, and, it already works and is up and running. The code I will detail before is targeted directly at optimizing images. If you do not already have a working gulpfile, I suggest
searching Google on setting up Gulp before reading this post. :)

First, make sure to add the following to your package.json dependencies section:

```json
"concurrent-transform": "1.0.0",
"gulp-changed": "1.3.1",
"gulp-image-resize": "0.8.0",
"gulp-imagemin": "3.0.1",
"gulp-webp": "2.3.0",
"imagemin-giflossy": "5.1.0-2",
"imagemin-mozjpeg": "6.0.0",
"imagemin-pngquant": "5.0.0",
"os": "0.1.1"
```

These plugins will help resize your images, compress them, and then output them to your destination directory.

>Warning: After adding the plugin declarations into your package.json, make sure to run `npm install`, otherwise, they will not work!

Then, make sure to add the following variable declarations towards the top of your gulpfile.js:

```js
// Loads gulp plugins using $.pluginname
var $ = require('gulp-load-plugins')();

// Parallelize Operations
var parallel = require('concurrent-transform');
var os       = require("os");

// Imagemin Plugins
var giflossy = require('imagemin-giflossy');
var mozjpeg  = require('imagemin-mozjpeg');
var pngquant = require('imagemin-pngquant');
var webp     = require('imagemin-webp');
```

Next, let's start adding some code! First, the resize images task. Let's start with the following task shell that comprises all gulp tasks:

```js
gulp.task('resize', () => {
    return gulp.src('_assets/images/**/*.{jpg,png}')
        .pipe(gulp.dest('public/tmp/responsive'));
});
```

Notice: gulp.src is the source directory. I am filtering all filetypes, with the exception of jpg and png (I don't need to resize svg files). gulp.dest is where I want the resized images to be outputted. In this case, I am outputting to a temporary directory. This will be used in the next task, and then removed.

At this point, the gulp task is doing nothing. It's pretty much just copying files from the src directory to the dst directory. Let's add some functionality:

```js
gulp.task('resize', () => {
    return gulp.src('_assets/images/**/*.{jpg,png}')
        .pipe($.changed('public/assets/images')) // <-- NEW
        .pipe(gulp.dest('public/tmp/responsive'));
});
```

_Changed_: Check to see if files in dst is newer than src. This means that this task will only "pipe" or "process" images that were added since the last run! That means, no need to reprocess hundreds or thousands of images each
time. Awesome! When adding changed, always set it to the dst of the gulp task. However, if you look closely, mine is set to a different dst directory? That's because that directory is the "true" final dst directory, and not the
temp directory. Otherwise, it will resize on every single run, since I delete the temp directory in the end hehe.

Now, we add the actual resize task. Excited yet???

```js
gulp.task('resize', () => {
    return gulp.src('_assets/images/**/*.{jpg,png}')
        .pipe($.changed('public/assets/images'))
        .pipe(parallel(
            $.imageResize({
                width : 1000,
                crop : false,
                quality: 1,
                upscale : false
            }),
            os.cpus().length
        ))
        .pipe(gulp.dest('public/tmp/responsive'));
});
```

Wow, a lot is going on in the meat of the task. Let's break it down, shall we?

_Parallel_: Run task with concurrency on files in the pipe. The way I am using it here is, I am using `os.cpus().length` to determine the amount of CPU cores, and run enough threads equal to the amount of CPUs. This doesn't necessarily guarantee a task for each CPU, but, at least you can take advantage of some concurrency. Therefore, you can see that parallel() takes two arguments: `parallel(function, amountOfThreads)`.

_$.imageResize_: Run resize task on all images in pipe. As you can see, I pass a couple of options to the function. I force all my images to have a max _width_ of 1000px, no _crop_, keep _quality_ as close to lossless as possible (I will reduce quality in my next task), and do not _upscale_ (keep smaller images smaller).

Now that we've built the resize task, we can move onto our next two tasks, image compression!

```js
// Minify and Compress images (webp not supported yet)
gulp.task('images', ['resize'], () => {
    return gulp.src(['_assets/images/**/*.{gif,svg,tiff}', 'public/tmp/responsive/**/*.{jpg,png}'])
        .pipe($.changed('public/assets/images'))
        .pipe(gulp.dest('public/assets/images'));
});

// Minify and Compress images -- outputting webp files
gulp.task('webp', ['resize'], () => {
    return gulp.src(['_assets/images/**/*.{tiff}', 'public/tmp/responsive/**/*.{jpg,png}'])
        .pipe($.changed('public/assets/images'))
        .pipe(gulp.dest('public/assets/images'));
});
```

Again, we start with the skeleton (and include `$.changed`, which you should already be familiar with) for both tasks, notice the similarity? Once you get the hang of creating tasks in Gulp, it's super easy to define anything, really! Also notice how these `$.changed` dst directories are point to the gulp.dst directories, like the `$.changed` in our resizing task! That way, all of them are monitoring the actual output directory, not temp ones.

Take a look at something cool I am doing with this tasks: I am only running them, if the resize task completes successfully. You can imagine why, how can I optimize resized images, if the images task runs while the resize task is running. Not a good situation to be in. Using the following syntax in a task `gulp.task('webp', ['resize'], () => {`, with the brackets like so `['resize']`, ensures the task will only run once the resize task completes. Very useful indeed.

```js
// Minify and Compress images (webp not supported yet)
gulp.task('images', ['resize'], () => {
    return gulp.src(['_assets/images/**/*.{gif,svg,tiff}', 'public/tmp/responsive/**/*.{jpg,png}'])
        .pipe($.changed('public/assets/images'))
        .pipe(parallel(
            $.imagemin([
                $.imagemin.gifsicle({interlaced: true}),
                $.imagemin.jpegtran({progressive: true}),
                $.imagemin.optipng(),
                $.imagemin.svgo({svgoPlugins: [{removeViewBox: false}]}),
                giflossy({optimizationLevel: 3, lossy: 80}),
                mozjpeg({quality: '70'}),
                pngquant({quality: '70-80'})
            ]),
            os.cpus().length
        ))
        .pipe(gulp.dest('public/assets/images'));
});

// Minify and Compress images -- outputting webp files
gulp.task('webp', ['resize'], () => {
    return gulp.src(['_assets/images/**/*.{tiff}', 'public/tmp/responsive/**/*.{jpg,png}'])
        .pipe($.changed('public/assets/images'))
        .pipe(parallel(
            $.webp({quality: 50, alphaQuality: 50}),
            os.cpus().length
        ))
        .pipe(gulp.dest('public/assets/images'));
});
```

We added some new lines, but again, you should already be used to the nomenclature, and feel familiar with the additions. :) In this case, we are using the plugins we added earlier (gulp versions included with imagemin
referenced using `$.`; ones not auto-loaded referenced using our variable declarations mentioned earlier), and the gulp-webp plugin in our webp task (imagemin's webp plugin doesn't rename files, I may do a PR to fix this
when I have some time). Then, I set some options regarding quality in the plugin declarations.

At this point, as long as you adjust the src and dst paths within the code in this post, it will work (barring any freaky circumstances, and not following my warning of `npm install` mentioned earlier). However, for thoses who just want the code, please copy/paste the code mentioned in this post from the TL;DR section below!


### TL;DR ###

>Warning: The following code assumes you already have Gulp up-and-running, and are pasting in a working install of Gulp. If this does not describe you, well, you may find the following code will not work for you.... In that case, stay tuned. I'll have a getting up-and-running with Gulp post soon. ;)

_package.json_

```json
"concurrent-transform": "1.0.0",
"gulp-changed": "1.3.1",
"gulp-image-resize": "0.8.0",
"gulp-imagemin": "3.0.1",
"gulp-webp": "2.3.0",
"imagemin-giflossy": "5.1.0-2",
"imagemin-mozjpeg": "6.0.0",
"imagemin-pngquant": "5.0.0",
"os": "0.1.1"
```

_gulpfile.js_

```js
// Loads gulp plugins using $.pluginname
var $ = require('gulp-load-plugins')();

// Parallelize Operations
var parallel = require('concurrent-transform');
var os       = require("os");

// Imagemin Plugins
var giflossy = require('imagemin-giflossy');
var mozjpeg  = require('imagemin-mozjpeg');
var pngquant = require('imagemin-pngquant');
var webp     = require('imagemin-webp');

gulp.task('resize', () => {
    return gulp.src('_assets/images/**/*.{jpg,png}')
        .pipe($.changed('public/assets/images'))
        .pipe(parallel(
            $.imageResize({
                width : 1000,
                crop : false,
                quality: 1,
                upscale : false
            }),
            os.cpus().length
        ))
        .pipe(gulp.dest('public/tmp/responsive'));
});

// Minify and Compress images (webp not supported yet)
gulp.task('images', ['resize'], () => {
    return gulp.src(['_assets/images/**/*.{gif,svg,tiff}', 'public/tmp/responsive/**/*.{jpg,png}'])
        .pipe($.changed('public/assets/images'))
        .pipe(parallel(
            $.imagemin([
                $.imagemin.gifsicle({interlaced: true}),
                $.imagemin.jpegtran({progressive: true}),
                $.imagemin.optipng(),
                $.imagemin.svgo({svgoPlugins: [{removeViewBox: false}]}),
                giflossy({optimizationLevel: 3, lossy: 80}),
                mozjpeg({quality: '70'}),
                pngquant({quality: '70-80'})
            ]),
            os.cpus().length
        ))
        .pipe(gulp.dest('public/assets/images'));
});

// Minify and Compress images -- outputting webp files
gulp.task('webp', ['resize'], () => {
    return gulp.src(['_assets/images/**/*.{tiff}', 'public/tmp/responsive/**/*.{jpg,png}'])
        .pipe($.changed('public/assets/images'))
        .pipe(parallel(
            $.webp({quality: 50, alphaQuality: 50}),
            os.cpus().length
        ))
        .pipe(gulp.dest('public/assets/images'));
});
```
