# SIMPLE BIG THEME

Contents:
1. Theming in general (and making theming faster)
2. What does the SIMPLE BIG THEME do


## THEMING IN GENERAL (AND MAKING THEMING FASTER)
Theming has long been a challenging, time-consuming task. Normally, it consists of finding somEthing you want to change, tracing that element to it's SASS rules in edx-platform, recreating and changing the rule in your theme's sass files, then re-compiling assets. The asset compilcation process takes about 10 minutes on a fast server.

This document attempts to de-mistify the process (just a tiny bit) and provide some extra options to do certain tasks faster.

### Asset Compilation the old-fashioned way

This command will compile the lms assets for a particular theme. It takes about 10 minutes.

        > paver update_assets lms --settings=production --themes=my_theme


### The two-step, faster way

If you are working with just the HTML and SASS templates (JS untested at this point), which we do 99% of the time, then you can use a two-step process that seems much faster.

First, This command creates the css files inside of the theme directory (e.g. themes/my_theme/lms/static/css)
        > python manage.py lms --settings=production compile_sass lms  --themes my_theme

Takes about 30-40 seconds with a small theme

If the task fails, it will leave partial versions of the sass files in the theme directory (e.g. lms-main-v1.12weoij211.sass). It probably does this as a way to trace your errors, but you can clean up your theme by fixing the errors and running it again.

Second, this command moves the compiled css files from the theme directory to the final /edx/var/edxapp/staticfiles/my_theme/css location.
        > python manage.py lms --settings=production collectstatic --noinput

Takes about 90 seconds with a small theme

It appends a random string (?) to the end of the files. Presumably for some sort of security, scaling, caching requirement, but I don't know. I also don't know how that string is determined or how LMS/CMS knows which version of the files to use.
Note that this does not seems to take a --themes argument

### The CSS Override method
You can also create a simple CSS file with your overrides. Your LMS will reference this file last, so it will act truly as an override. Also, updates to this file don't need to be compiled or two-step compiled. They are instant!
The process below is still a rough one. It looks like EdX at one point had a supported way of doing this, but their documentation around it has gone out of date.

1. Create your CSS file somewhere in your theme folder. e.g. my-theme/lms/static/css_overrides/my-css.css
2. Create a symbolic link (soft) in  /edx/var/edxapp/staticfiles/my_theme/css_overrides/my-css.css (or something similar). We have to create this link outside of the theme because this is where Edx goes looking for CSS files.
3. Add the head-extra.html file to my-theme/lms/templates. (This readme has some explanation of the head-extra.html file: https://github.com/edx/edx-platform/blob/master/themes/README.rst)
4. The head-extra.html wants to look for static_content.html so you have to copy that from edx-platform repo and place it in your theme where it can be referred by head-extra.html (by default, this is two directories up)
        NOTE: This seems like a bug. I would rather keep static_content.html in edx-platform and refer to it there, since I don't change it. But I hit errors when trying to accomplish this.
5. Modify the head-extra.html file to refer to the CSS file you created in Step 1.
        Note: i couldn't get this to work with the if style_overrides_file conditional, so I just created a new line outside the conditional
        My line is <link rel="stylesheet" type="text/css" href="/static/curricume_clean/css_overrides/my-css.css" />

If you fail at any step, the logs do a pretty good job of telling you where it is failing (usually a file reference is broken) and leading you to a fix.

## WHAT DOES THE SIMPLE BIG THEME DO?
The SIMPLE BIG THEME acts as a template to demonstrate several theming methods.

- It overrides EdX default SASS variables to dictate colors
- It imports its own custom SASS files to override the default EdX theme
- It utilizes simple CSS files for additional overrides (e.g. to the homepage header)
- It adds and overrides some images
- It overrides the homepage with a custom header.
- It utilizes static templates (e.g. for About and Contact pages)

### Overrides EdX default SASS variables to dictate colors. 
Variables are defined in a few places in the default edx theme. The most notable location as of Koa seems to be  lms/static/sass/partials/lms/theme. But EdX seems to have been moving this around recently, so I'm not always clear on it. In SIMPLE BIG THEME, we override EdX default variables (i.e. $primary and $inverse) in _variables.scss. So we create our own version of _variables.scss in our theme and write our overrides there. We do it here because SASS takes the first variable declaration it finds.
EdX documentation seems to make it seem like _extra.scss is the place for these variable overrides, but _extra.scss is compiled near last in the process. So the variable names by that time are already defined and can't be replaced. So _extra.scss could be a place to create brand new variables that are only used in subsequent custom sass files, but it is not the place to override EdX's defaults. 

### Import its own custom SASS files to override the default EdX theme
As opposed to variables, which you want to be first in line for, with SASS customizations you want to be last in line. This way, the compiled CSS will be last and therefore have a higher priority. So now we can use our my-theme/lms/static/sass/partials/lms/theme/_extras.scss file and add some lines to call our custom SASS files from here.
It is not important at this point to match the structure of edx-platform. You just have to make sure your SASS rules will overwrite the default rules. You very well may have a file at my-theme/lms/static/sass/partials/lms/theme/_my-footer.scss that overwrites the defaults in edx-platform/lms/static/sass/shared/_footer.scss

### Utilize simple CSS files for additional overrides (e.g. to the homepage header)
See "The CSS Override Method" for some brainstorm on why things are organized this way. Some of it, admittedly, seems either buggy or misunderstood by me.
I have a head-extra.html file in my templates folder. From this file, I can call any custom .css files. I edit my .css files in my theme but create a link to them in the /edx/var/edxapp/ area where all the final theme files are compiled and then referenced from.
Head-extra.html relies on another file called static_content.html file which is in my theme in the very top level.

### Add and override some images
This one is fairly simple to understand. If you place an image in my-theme/lms/static/images and that image has the same name as a corresponding imaage in default EdX, then your image will supersede the default.
If your image does not have a corresponding image, then you'll need to reference it in one of your html files in order for it to appear somewhere. That looks something like:
        > <img src="${static.url('images/my-img.png')}">

### Override the homepage with a custom header
On the homepage I want a header that has a little more information than the default Open EdX Header. I also may want to be able to customize this easily in the future, without having to deal with so much SASS and asset compilation.
This takes a couple steps
1. I create a custom HTML header called header-homepage.html
2. I need this custom header to only apply to my homepage. Since every page starts with main.html I have to make a special copy of main.html for my homepage. So I copy main.html to a new version called main-homepage.html. Open to better ideas here!
3. I then modify main-homepage.html. Where it calls for the header.html I add my custom header-homepage.html
        Note: You'll undoubtedly run into problems as you start creating and moving these files with some of the many relative references breaking, and your theme breaking. The logs are often a quick way to debug and fix these errors.
4. After making sure my homepage html file (index.html) is in my theme, I modify it to inherit my special main-homepage.html instead of the usual ma
in.html

At this point, my homepage (index.html) is calling my special template file (main-homepage.html) which is calling my special homepage-only header (header-homepage.html). In addition, my homepage header has it's own simple CSS file which uses the same method as described above. It is all working. 

###Utilize static templates
Static pages, like an About or Contact page, have a special place in the theme at my-theme/lms/templates/static_templates. 
For a page that already exists like about.html or tos.html, simply copy the html file into your theme and make your modifications.
For a page that does not exist (e.g. my-about.html), you will need to create the file in the static_templates directory and also update your MKTG_UR
L_LINK_MAP in your configuration file (e.g. lms.yml or lms.env.json). Make sure to restart lms and cms after you make any changes to your configurat
ion files.

