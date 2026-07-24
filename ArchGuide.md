# Arch guide

This guide is a simple walk through of what different things in Arch do. Some features were already used in the Install Guide, and should be familiar.

## Kernel

The linux kernel is what really is linux. Everything on top is packages and the Desktop Environment.

## DE

The Desktop Environment is what is sat ontop of the base Kernel. We downloaded the kernel with pacstrap. The DE is what we installed ontop, called KDE Plasma.

SDDM (Simple Desktop Display Manager) is what actually does all the displaying. KDE sits with SDDM to do all the cool desktop stuff.

Ontop of this, the window compositor sits. Wayland is the most common one used now, with X11 being more rarly used. Some others like hyprland is used by some crazy programming people.

## GRUB

GRUB is the boot manager for linux. Basically all modern linux distros use GRUB in place of anything else because it fucking works (and why bother re-inventing the wheel).

GRUB is smart, and not greedy like the windows boot manager, however, it can mess up with safe-boot on (since that itself is a Microsoft thing. Thank you Microslop.)

As seen in the install guide, we made a GRUB config for the Arch build. This means that we can have multiple configs if we wanted to. (Does anyone do this? Well, some weird people do)

Of course, since its our goat, GRUB allows for multiple EFIs to be pointed to, including windows.

## File space

As you saw in mounting the file to /mnt, we mount the EFI boot sector inside of the root directory. What does this do? It allows us to split the root directory with the EFI. If we did it the other way around, the BIOS/GRUB would have full access to literally everything on disk, which can be dangerous. (The less visible, the better)

On windows, we had disk names, like C: or D:. This does not exist on linux. Everything sits on / and ranks from there.

Fun fact: Literally everything on Linux is a file. Like literally.

If you want to know memory usage, its written in a file. If you want the CPU usage, its in a file.

These files do actually exist, but are usually read-only from userspace.

## Linux Terminal

The linux terminal as you know it, when opened as an app is not actually the terminal itself, but an emulation of the actual terminal (since SDDM covers the actual terminal). 

The important thing about this is that we can have different terminal emulators. The most common is bash and zsh. Does it really matter to have something other than bash? No. Should you have something other than bash? Eh? probably not.

There are also a couple of cool terminal emulators that allow you to have multiple terminal windows in one, but its not useful unless you are trying to make things in the actual linux terminal instead of a terminal emulator like Konsole.

### Linux Directory

Directories are important. They tell you where things are in file. Some things make it easier.

 - ~ \- The tilda points to $HOME for the current user.
 - . \- A single dot points to $PATH or current path.
 - .. \- A double dot points to the parent directory.

These are the main 3 directory shortcuts.

Some others are the
 - \* \- every file in under a directory

Just like any other OS, you can enter a sub-path from the CWD, instead of the absolute path.

The first three come at the start, the last one comes at the end. EG: ~/.config/* gives everything in the .config file under $HOME.

### Basic Terminal commands

Linux actually has quite a lot of similarities with Windows in terms of terminal commands.

 - cd - Change directory.
 - mv - Moves a file or directory to another. You can also rename files by entering a different file name.
 - ls - Lists all files in current directory
 - rm - Removes a file. (For a directory, add the -rf flag.) **This completely deletes the file. If you want to trash, use trash-cli**
 - mkdir - Makes a directory
 - touch - Makes a file
 - echo - returns text. It can be fed into a file using >>

The cd command alone will send you to $HOME. Sometimes using just cd is very difficult, so we use some other commands like below.

### More commands!

 - ranger - Added with pacman, allows you to open a Terminal based file explorer, kind of like the neo-tree. It uses vim movement and basic exit commands. Using it as (. ranger) allows you to move directories easily
 - trash - Added with pacman (trash-cli), allows you to remove files and directories and move them to trash instead of completely removing the file like with rm.
 - aria2c - our favorite terminal based torrenter.

## Coding

Most code compilations and executions should be done using the terminal! This makes life actually so much easier than fiddling with VSCode's extensions. Honestly, even if you keep using VSCode, I would highly suggest doing at least this.

### Python

Python only has an interpreter. There are tools to create executables, but this is usually avoided. Most people use python as a library creator for other languages.

```sh
python <dir>
```

On Arch, python libraries are not really installed using PIP. This is because python is installed using pacman, and pip voids the safety that Arch wants. The real way to install is through pacman. Usually they are named python-<library name>

Example:

```sh
sudo pacman -S python-pygame
```

Python is very easy for VSCode to run because it's just a simple command. This is why VSCode works really well for this. For anything else? uhhh not really that good.

### C / C++

Not sure how much you care, but I'm adding it just in case.

The modern C/C++ compiler is called gcc. C/C++ can be compiled to an executable or Assembly code. In both cases, we just call it:

```sh
gcc <file path> -o <output name>
```

Running the compiled file is almost easier! We just need to call the file directly.

```sh
./<executable name>
```

Executables in linux don't actually have any file type, unlike windows which uses .exe.

Example usage:

```sh
gcc test.c -o code
./code
```

### C#

Possibly the one thing that windows does better over linux is doing anything with C# (since they made it).

Because Microsoft is a bitch, they made C# kinda Windows exclusive, allowing it to only compile to .exe, which cannot be ran directly on linux. 

Once you somehow got dotnet working, we create a dotnet project, place the code inside, then build and release the project, and find the executable, which can be run. Very very fucking annoying.

```sh
dotnet new console -o <Project Name>
```

This creates the project, then you edit the files inside, writing C# code, then to run:

```sh
dotnet run
```

There should be a .sh file (bash file) that can or maybe cannot be ran (may need execution access)

```sh
chmod +x <.sh executable path>
```

and this absolute mess is why we all hate Microshit.

### Java

My favourite!

Java only needs to be compiled to .class and .jar files. Then it can be ran with "java"

```sh
javac <.java file path>
java <same file name without .java>
```

That was... so much easier wasn't it!

### Anything else

Most languages go through the Compile -> Execute step, using two commnads. This is why VSCode can struggle a lot. The other way is just Interpret, so it needs one command. Languages like C do sometimes come with external packages that has interpret, but I wouldn't usage them since the 2 command method is really simple.

Linux also allows you to combine 2 or more commands into one line using &&. Example:

```sh
gcc test.c -o test && ./test
```

This runs the test.c like an interpreter with one line! Of course, the errors come both at compile time and run time instead of both at run time like an intepreter would, but since we are using both on one line, it basically appears as one.


## Sudo

Sudo is the command called before another terminal command to call a function in root privaleges (basically Admin mode)

Usage:
```
sudo ...
```

Easy right?

## Package managers

Package managers are software that keep track of every installed module. Basically everything you install, unless directly installed from git is managed using package managers.

### Pacman

Pacman is the main package manager for Arch. This is used to install mostly anything that is base in linux, and quite a lot of other things.

Basic usage is
```sh
sudo pacman -S ...
```

the -S is called a flag, usually coming in two forms. The first is with a single - and the other is with --. By convention, - is used before a single character abbreviation of the full -- flag.

The flags available for pacman are visible by doing pacman -h.

The common ones that are used are:

 - -S - install
 - -Sy - clean install
 - -Syu - clean install and update everything else

 - -R - uninstall (remove)
 - -Rns - uninstall removing all unused dependencies **This one is usually the most optimal one to use.**

Realistically, you will only use -S -Syu and -Rns.

### YAY

yay is a user repository package manager. This means its a little more open ended, which could be more dangerous, but is managed by a good team. It takes from user gits.

it uses basically the same flags as pacman (since they are both arch specific).

When installing, you can ignore most things and just press *enter* until you reach the end.

### flatpak

Flatpak is an application manager. This differs from pacman and yay since they usually install packages and command-line based applications, but flatpak specialises in managing UI based applications.

Most distros and DE come pre-packaged with flatpak. This includes our KDE install process.

Flatpak installs files into a raised location, which can sometimes not access certain things, and require flatseal, a separate application that fixes this issue.

## Wine

Mmm tasty... But more importantly, wine is a windows .exe runner for linux (like a VM). Wine can run .exe files! Except that doesn't mean we can download and use windows apps, since wine contains all activity, and the .exe files you get for installers are built for windows, and will catastrophically fail. Not that it matters for most applications, but if you have a *portable* version of the app, you can run with wine. 

## Git

You are probably more used to github, but that is only a website that can host git repositories. A git repository is basically just a cool google drive, which has neat features like:

### Clone

A git clone basically is just a download of the repository to your own machine.

The command is
```sh
git clone ...
```

Most of what you will do is cloning and building using makepkg. 

### Fork

Forking is a git feature that basically makes a copy of the original in a separate repository that holds the ability to upgrade. Kind of like polymorphism in OOP.

### Push/Pull

Pushing and Pulling are actions on a git repository that adds or removes files from them. Usually there is a main repo, and different branches that are updated simultaneously by different people, and tested separately. When the owner is satisfied with the changes, the branches are merged into the main branch.


## More important differences between linux and windows!

Everything above is very pedantic in terms of what is being said. Here, we will have to go through some important differences in terms of how we use the computer.

### Switching tabs

All tab switching is done with the ALT + num key combination instead of CTRL + num.

### Super key

The "Super" key is often seen in the settings for setting custom keybinds (yes you can change base keybinds for things. Thank you linux.), the super key is just the windows key. It is the same key, re-utilised for linux.

A lot of the same things is done using the super key.

### Runner

A runner is, as the name suggests, something that can run a simple command, open apps and so on. A stronger version of the search in the applications menu opened by pressing the Super button. It is (by default) ALT + Space. 

In KDE, this is also accessible through the window view menu, which can be opened with Super + w.

Use this, because its really good. It can even find files just here and there.

### Searching files in Dolphin

Dolphin is the main file manager that most people use on linux. It is pretty good, but most of all, the file finding (grep) is crazy good. Use it.

### Customisation!

Windows is... to put it lightly, very very very very lacking in the customisation department. And adding new applications just to be able to customise? Very annoying. Linux? Customisation heaven. Maybe it is a little daunting with how much customisation is available.

So, what makes this process a little easier? Well, the community! KDE offers a variety of stores for each of the different parts of the UI, including (but not all) the desktop theme, window theme, icon theme, etc.

All of this can be found in system settings. Honestly the fun of running KDE is the system settings. Also never forget the strength that Window Management and Desktop Effects have. They can be crazy fun and makes everything look *clean*.

### Downsides...

Well, with all good things, comes the bad. There are a lot of things missing from linux, especially the compatability with certain apps, such as adobe and microsoft365 apps or something like paint.net and similar. 

There are alternatives to all of them, however, they usually lack features, are less supported, and they fail to deliver. (At least they are free right?)

## How to fix things

Ask gemini... I've literally fixed all my problems by throwing it to gemini (not even the thinking model, just the fast model)

But don't just blinding follow what it tells you, use what it tells you, understand what its telling you.

## Common Arch issues and fixes:

Yes, there are many problems that do come with arch, some that just seems non-sensical, but are actually not that bad.

### No WiFi

If you got to this situation before getting a Terminal Emulator, you are... Not fucked! Just run the following in the runner! (Or terminal if you did get it)

```sh
systemctl start networkmanager
```

If this throws a "does not exist" error, we have to go back to our linux installer, mount, change root, then add Network Manager again, using pacman.

### No bluetooth

Well yes, of course. We didn't install bluetooth!

```sh
sudo pacman -S bluez bluez-utils
sudo systemctl enable bluetooth
sudo systemctl start bluetooth
```

This should immediately allow you to add things from system settings.

