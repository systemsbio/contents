# How To Use Makefiles to Automate Repetitive Tasks on an Ubuntu VPS

 

Modified from 
https://www.digitalocean.com/community/tutorials/how-to-use-makefiles-to-automate-repetitive-tasks-on-an-ubuntu-vps
By Justin Ellingwood
This work is licensed under a Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License. 

### Introduction

* * *

If you have experience installing software from source code on your Linux server, you have probably come across the `make` utility. This tool is primarily used to automate the compilation and building of programs. It allows the application’s author to easily lay out the steps required to build that specific project.

Although make was created to automate software compilation, the tool was engineered in a flexible enough fashion that it can be used to automate almost any task that you can accomplish from the command line. In this guide, we will discuss how you can repurpose make to automate repetitive tasks that happen in sequence.

We will be demonstrating this on an Ubuntu 12.04 VPS, but it should operate in a similar manner on almost any Linux server.

<a name="install-make" data-unique="install-make"></a><a name="install-make" data-unique="install-make"></a>

## Install Make

* * *

Before we begin using make, we need to install it.

Although we can install it by name, it is usually installed along with other tools that help you compile software. We will install all of these because they are incredibly useful to have in general. There is a package called “build-essential” that contains make and these other programs:

    sudo apt-get update
    sudo apt-get install build-essential

Now you have the tools that will allow you to take advantage of make in its usual capacity as well.

<a name="understanding-makefiles" data-unique="understanding-makefiles"></a><a name="understanding-makefiles" data-unique="understanding-makefiles"></a>

## Understanding Makefiles

* * *

The primary way that the make command receives instructions is through the use of a `Makefile`.

From the manual page, we can see that make will look for a file named GNUmakefile, and then makefile, and then Makefile. It recommends that you use Makefile, since that the GNUmakefile is for GNU-specific commands, and makefile does not stand out as much.

Makefiles are directory specific, meaning that make will search in the directory where it was called to find these files. As such, we should put the Makefile in the root of whatever the task that we are going to be performing, or where it makes most sense to call the scripts we will write.

Inside the Makefile, we follow a specific format. Make uses the concepts of targets, sources, and commands in this way:

    target: source
        command

The alignment and format of this is very important, because make is picky. We will discuss the format and meaning of each of these components here:

### Target

* * *

The target is a user-specified name to refer to a group of commands. Think of it as similar to a simple function in a programming language.

A target is aligned on the left-hand column, is a continuous word (no spaces), and ends with a colon (:).

When calling make, we can specify a target by typing:

<pre>make target_name
</pre>

Make will then check the Makefile and execute the commands associated with that target.

### Source

* * *

Sources are references to files or other targets. They represent prerequisites or dependencies for the target they are associated with.

For instance, you could have a section of your make file that looks like this:

    target1: target2
        target1_command
    
    target2:
        target2_command

In this example, we could call target1 like this:

    make target1

Make would then go to the Makefile and search for the “target1” target. It would then check to see if there are any sources specified.

It would find the “target2” source dependency and jump temporarily to that target.

From there, it would check if target2 had any sources listed. It does not, so it would proceed to execute “target2_command”. At this point, make would reach the end of the “target2” command list and pass control back to the “target1” target. It would then execute “target1_command” and exit.

Sources can be either files or targets themselves. Make uses file timestamps to see if a file has been changed since its last invocation. If a change to a source file has been made, that target is re-run. Otherwise, it marks the dependency as fulfilled and continues to the next source, or the commands if that was the only source.

The general idea is that by adding sources, we can build a sequential set of dependencies that must be executed before the current target. You can specify more than one source, separated by spaces, after any target. You can begin to see how you can specify elaborate sequences of tasks.

### Commands

* * *

What gives the make command such flexibility is that the command portion of the syntax is very open. You can specify any command you wish to run under the target. You can add as many commands as you would like.

Commands are specified on the line after the target declaration. They are indented by one **tab** character. Some versions of make are flexible about the way that you indent the command section, but in general, you should stick with a single tab to ensure that make will recognize your intent.

Make considers every indented line under the target definition to be a separate command. You can add as many indented lines and commands as you would like. Make will simply go through them one at a time.

There are a few things that we can place before commands to tell make to handle them differently:

*   **-**: A dash before a command tells make to **not** abort if an error is encountered. For instance, this could be useful if you want to execute a command on a file if it is present, and do nothing if it is not.

*   **@**: If you lead a command with the “@” symbol, the command call itself will not be printed to standard output. This is mainly used just to clean up the output that make produces.

<a name="additional-features" data-unique="additional-features"></a><a name="additional-features" data-unique="additional-features"></a>

## Additional Features

* * *

Some additional features can help you create more complex rule chains in your Makefile.

### Variables

* * *

Make recognizes variables (or macros), which behave as simple placeholders for substitution in your makefile. It is best to declare these at the top of your file.

The name of each variable is completely capitalized. Following the name, an equal sign assigns the name to the value on the right side. For instance, if we wanted to define an installation directory as `/usr/bin`, we could add this at the top of the file:

    INSTALLDIR=/usr/bin

Later in the file, we can reference this location by using this syntax:

    $(INSTALLDIR)

### Escape Newlines

* * *

Another useful thing that we can do is allow commands to span multiple lines.

We can use any command or shell functionality within the command section. This includes escaping newline characters by ending the line with “":

    target: source
        command1 arg1 arg2 arg3 arg4 \
        arg5 arg6

This becomes more important if you take advantage of some of the shell’s more programmatic functionality, like if-then statements:

    target: source
        if [ "condition_1" == "condition_2" ];\
        then\
            command to execute;\
            another command;\
        else\
            alternative command;\
        fi

This will execute this block as if it were a one line command. In fact, we could have written this as one line, but it improves readability considerably breaking it down like this.

If you are going to escape end of line characters, be certain to not have any extra spaces or tabs after the ”", or else you will get an error.

### File Suffix Rules

* * *

An additional feature that you can use if you doing file processing is file suffixes. These are general rules that provide a way of processing files based on their extension.

For instance, if you want to process all .jpg files in a directory and convert them into .png files using the ImageMagick suite, we could have something like this in our Makefile:

    .SUFFIXES: .jpg .png
    
    .jpg.png:
        @echo converting $< to $@
        convert $< $@

There are a few things we need to look at here.

The first part is the `.SUFFIXES:` declaration. This tells make about all of the suffixes we will use in file suffixes. Some suffixes that are used often in compiling source code, like “.c” and “.o” files are included by default and do not need to be labeled in this declaration.

The next part is the declaration of the actual suffix rule. This basically takes the form of:

<pre>original_extension.target_extension:
</pre>

This is not an actual target, but it will match any call for files with the second extension and build them out of the file in the first extension.

In our case, we can call make like this to build a file called “file.png” if there is a “file.jpg” in our directory:

    make file.png

Make will see the png file in the `.SUFFIXES` declaration and see the rule for creating “.png” files. It will then look for the target file with the “.png” replaced by “.jpg” in the directory. It will then execute the commands that follow.

The suffix rules use some variables that we have not been introduced to yet. These help make substitute different information based on what part of the process it is currently in:

*   **$?**: This variable contains the list of dependencies for the current target that are more recent than the target. These would be the targets that must be re-done before executing the commands under this target.

*   **$@**: This variable is the name of the current target. This allows us to reference the file you are trying to make, even though this rule was matched through a pattern.

*   **$<**: This is the name of the current dependency. In the case of suffix rules, this is the name of the file that is used to create the target. In our example, this would contain “file.jpg”

*   **$***: This file is the name of the current dependency with the matched extension stripped off. Consider this an intermediate stage between the target and source files.

<a name="create-a-conversion-makefile" data-unique="create-a-conversion-makefile"></a><a name="create-a-conversion-makefile" data-unique="create-a-conversion-makefile"></a>

## Create A Conversion Makefile

* * *

We will create a Makefile that will do some image manipulations and then upload the files to our file server, so that our website can then display them.

If you would like to follow along, before you begin, download the ImageMagick conversion tools. These are easy command line tools for manipulating images and we will make use of them in our script:

    sudo apt-get update
    sudo apt-get install imagemagick

In your current directory, create a file called `Makefile`:

    nano Makefile

Within this file, we will start implementing our conversion targets.

### Convert all JPG Files to PNG

* * *

Our server has been set up to serve .png images exclusively. Because of this, we need to convert any .jpg files to .png before uploading.

As we learned above, a suffix rule is a great way of doing this. We will start out with the `.SUFFIX` declaration that will list the formats we are converting between:

    .SUFFIXES: .jpg .png

Afterwards, we can make a rule that will change .jpg files into .png files. We can do this using the `convert` command from the ImageMagick suite. The convert command is simple, and its syntax is:

<pre>convert from_file to_file
</pre>

To implement this command, we need the suffix rule that specifies the format we’re starting with and the format we’re ending with:

    .SUFFIXES: .jpg .png
    
    .jpg.png:           ## This is the suffix rule declaration

Now that we have the rule that will match, we need to implement the actual convert step.

Because we don’t know exactly what filename will be matched here, we need to use the variables that we learned about. Specifically, we need to reference `$<` as the original file, and `$@` as the file we are converting to. If we combine this with what we know about the convert command, we get this rule:

    .SUFFIXES: .jpg .png
    
    .jpg.png:
        convert $< $@

Let’s add some functionality so that we can be told explicitly what is happening with an echo statement. We will include the “@” symbol before the new command and the command we already had in order to silence the actual command from being printed when it is executed:

<pre>.SUFFIXES: .jpg .png

.jpg.png:
    [@echo](https://www.digitalocean.com/community/users/echo) converting $< to $@ using ImageMagick...
    [@convert](https://www.digitalocean.com/community/users/convert) $< $@
    [@echo](https://www.digitalocean.com/community/users/echo) conversion to $@ successful!
</pre>

At this point, we should save and close the file so that we can test it.

Get a jpg file into the current directory. If you don’t have a file on hand, you can get one from the DigitalOcean website by typing:

    wget https://digitalocean.com/assets/v2/badges/digitalocean-badge-blue.jpg
    mv digitalocean-badge-blue.jpg badge.jpg

You can test whether your make file is working thus far by asking it to create a `badge.png` file:

    make badge.png

* * *

    converting badge.jpg to badge.png using ImageMagick...
    conversion to badge.png successful!

Make will go to the Makefile, see the .png in the `.SUFFIXES` declaration and then go to the suffix rule that matches. It then runs the commands listed.

### Create a File List

* * *

At this point, make can create a .png file if we explicitly tell it that we want that file.

It would be better if it just created a list of .jpg files in the current directory and then converted those. We can do this by creating a variable that holds all of our files to be converted.

The best way to do this is with the wildcard directive:

    JPG_FILES=$(wildcard *.jpg)

We could just specify a target with a bash wildcard like this:

    JPG_FILES=*.jpg

But this has a shortcoming. If there are no .jpg files, this actually tries to run the conversion commands on a file called “*.jpg”, which will fail.

The wildcard syntax we mentioned above compiles a list of .jpg files in the current directory, and if none exist, it doesn’t set the variable to anything.

While we are doing this, we should try to handle a slight variation in .jpg files that is common. These image files are often seen with the .jpeg extension instead of .jpg. To handle these in a sane way, we can change their name in our program to .jpg files, so that our processing lines are simpler:

Instead of the above lines, we will use these two:

    JPEG=$(wildcard *.jpg *.jpeg)     ## Has .jpeg and .jpg files
    JPG=$(JPEG:.jpeg=.jpg)            ## Only has .jpg files

The first line compiles a list of .jpg and .jpeg files in the current directory and stores them in a variable called `JPEG`.

The second line references this variable and does a simple name translation to convert the names in the `JPEG` variable that end with .jpeg into names that end with .jpg. This is done with this syntax:

    $(VARNAME:.convert_from=.convert_to)

At the end of these two lines, we will have a new variable called `JPG` which contains only .jpg filenames. Some of these files may not actually exist on the system, because they are actually .jpeg files (no actual renaming took place). This is okay because we are only using this list to make a _new_ list of .png files we want to create:

    JPEG=$(wildcard *.jpg *.jpeg)
    JPG=$(JPEG:.jpeg=.jpg)
    PNG=$(JPG:.jpg=.png)

Now, we have a list of files we want to request in the variable `PNG`. This list contains only .png filenames, because we did another name conversion. Now, every file that was a .jpg or .jpeg file in this directory has been used to compile a list of .png files we want to create.

We also need to update the `.SUFFIXES` declaration and the suffixes rule to reflect that we are now handling .jpeg files:

    JPEG=$(wildcard *.jpg *.jpeg)
    JPG=$(JPEG:.jpeg=.jpg)
    PNG=$(JPG:.jpg=.png)
    .SUFFIXES: .jpg .jpeg .png
    
    .jpeg.png .jpg.png:
        @echo converting $< to $@ using ImageMagick...
        @convert $< $@
        @echo conversion to $@ successful!

As you can see, we have added the .jpeg to the suffixes list and also included another suffix match for our rule.

### Create Some Targets

* * *

We have quite a lot in our Makefile right now, but we don’t have any normal targets yet. Let’s fix that so that we can pass our `PNG` list to our suffix rule:

<pre>JPEG=$(wildcard *.jpg *.jpeg)
JPG=$(JPEG:.jpeg=.jpg)
PNG=$(JPG:.jpg=.png)
.SUFFIXES: .jpg .jpeg .png

convert: $(PNG)

.jpeg.png .jpg.png:
    [@echo](https://www.digitalocean.com/community/users/echo) converting $< to $@ using ImageMagick...
    [@convert](https://www.digitalocean.com/community/users/convert) $< $@
    [@echo](https://www.digitalocean.com/community/users/echo) conversion to $@ successful!
</pre>

All this new target does is list the .png filenames that we gathered as a requirement. Make then sees if there’s a way that it can acquire the .png files and uses the suffix rule to do so.

Now, we can simply use this command to convert all of our .jpg and .jpeg files to .png files:

    make convert

Let’s add another target. Another task that is generally done when uploading images to a server is to resize them. Having your images the correct size will safe your users from having to resize images on the fly when they request them.

An ImageMagick command called `mogrify` can resize images in the way that we need. Let’s say the area where our images will be displayed on our site is 500px wide. We can convert for this area with the command:

    mogrify -resize 500\> file.png

This will resize any images larger than 500px wide to fit this area, but will not touch smaller images. This is what we want. As a target, we can add this rule:

<pre>resize: $(PNG)
    [@echo](https://www.digitalocean.com/community/users/echo) resizing file...
    [@mogrify](https://www.digitalocean.com/community/users/mogrify) -resize 648\> $(PNG)
    [@echo](https://www.digitalocean.com/community/users/echo) resizing is complete!
</pre>

We can add this to our file like this:

<pre>JPEG=$(wildcard *.jpg *.jpeg)
JPG=$(JPEG:.jpeg=.jpg)
PNG=$(JPG:.jpg=.png)
.SUFFIXES: .jpg .jpeg .png

convert: $(PNG)

resize: $(PNG)
    @echo resizing file...
    @mogrify -resize 648\> $(PNG)
    @echo resizing is complete!

.jpeg.png .jpg.png:
    [@echo](https://www.digitalocean.com/community/users/echo) converting $< to $@ using ImageMagick...
    [@convert](https://www.digitalocean.com/community/users/convert) $< $@
    [@echo](https://www.digitalocean.com/community/users/echo) conversion to $@ successful!
</pre>

Now, we can string these two targets together as dependencies of another target:

<pre>JPEG=$(wildcard *.jpg *.jpeg)
JPG=$(JPEG:.jpeg=.jpg)
PNG=$(JPG:.jpg=.png)
.SUFFIXES: .jpg .jpeg .png

webify: convert resize

convert: $(PNG)

resize: $(PNG)
    [@echo](https://www.digitalocean.com/community/users/echo) resizing file...
    [@mogrify](https://www.digitalocean.com/community/users/mogrify) -resize 648\> $(PNG)
    [@echo](https://www.digitalocean.com/community/users/echo) resizing is complete!

.jpeg.png .jpg.png:
    [@echo](https://www.digitalocean.com/community/users/echo) converting $< to $@ using ImageMagick...
    [@convert](https://www.digitalocean.com/community/users/convert) $< $@
    [@echo](https://www.digitalocean.com/community/users/echo) conversion to $@ successful!
</pre>

You may notice that resize implicitly will run the same commands as convert. We are going to specify them both though in case that is not always the case. The convert could in the future contain more elaborate processing.

The webify target now converts and resizes images.

### Upload Files to Remote Server

* * *

Now that we have our images ready for the web, we can create a target to upload them to the static images directory on our server. We can do this by passing our list of converted files to `scp`:

Our target will look something like this:

<pre>upload: webify
    scp $(PNG) root@ip_address:/path/to/static/images
</pre>

This will upload all of our files to the remote server. Our file now looks like this:

<pre>JPEG=$(wildcard *.jpg *.jpeg)
JPG=$(JPEG:.jpeg=.jpg)
PNG=$(JPG:.jpg=.png)
.SUFFIXES: .jpg .jpeg .png

upload: webify
    scp $(PNG) root@ip_address:/path/to/static/images

webify: convert resize

convert: $(PNG)

resize: $(PNG)
    [@echo](https://www.digitalocean.com/community/users/echo) resizing file...
    [@mogrify](https://www.digitalocean.com/community/users/mogrify) -resize 648\> $(PNG)
    [@echo](https://www.digitalocean.com/community/users/echo) resizing is complete!

.jpeg.png .jpg.png:
    [@echo](https://www.digitalocean.com/community/users/echo) converting $< to $@ using ImageMagick...
    [@convert](https://www.digitalocean.com/community/users/convert) $< $@
    [@echo](https://www.digitalocean.com/community/users/echo) conversion to $@ successful!
</pre>

### Clean Up

* * *

Let’s add a cleaning option to get rid of all of the local .png files after they have been uploaded to the remote server:

<pre>clean:
    rm *.png
</pre>

Now, we can add another target at the top, that calls this one after we upload our files to the remote server. This will be the most complete target, and the one that we want to be default.

To specify this, we will put it as the first target available. This will be used as the default. We will call it “all” by convention:

<pre>JPEG=$(wildcard *.jpg *.jpeg)
JPG=$(JPEG:.jpeg=.jpg)
PNG=$(JPG:.jpg=.png)
.SUFFIXES: .jpg .jpeg .png

all: upload clean

upload: webify
    scp $(PNG) root@ip_address:/path/to/static/images

webify: convert resize

convert: $(PNG)

resize: $(PNG)
    [@echo](https://www.digitalocean.com/community/users/echo) resizing file...
    [@mogrify](https://www.digitalocean.com/community/users/mogrify) -resize 648\> $(PNG)
    [@echo](https://www.digitalocean.com/community/users/echo) resizing is complete!

clean:
    rm *.png

.jpeg.png .jpg.png:
    [@echo](https://www.digitalocean.com/community/users/echo) converting $< to $@ using ImageMagick...
    [@convert](https://www.digitalocean.com/community/users/convert) $< $@
    [@echo](https://www.digitalocean.com/community/users/echo) conversion to $@ successful!
</pre>

With these last touches, if you enter the directory with the Makefile and .jpg or .jpeg files, you can call make without any arguments to process your files, send them to your server, and then delete the .png files you uploaded.

    make

As you can see, it is easy to string together tasks and also to cherry pick a process up to a certain point. For instance, if you only want to convert your files and need to host them on a different server, you can just use the webify target.

<a name="conclusion" data-unique="conclusion"></a><a name="conclusion" data-unique="conclusion"></a>

## Conclusion

* * *

At this point, you should have a good idea of how to use Makefiles in general. More specifically, you should know how to use make as a tool for automating most kinds of procedures.

While in some cases it may be easier to write a simple script, Makefiles are a simple way of setting up a structured, hierarchical relationship between processes. Learning how to leverage this tool can help make repetitive tasks simple.
