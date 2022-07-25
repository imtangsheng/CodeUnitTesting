https://flet.dev/docs/

https://github.com/flet-dev/flet


- [Introduction](#introduction)
- [Packaging a desktop app​](#packaging-a-desktop-app​)

## Introduction

What is Flet
Flet is a framework that allows building interactive multi-user web, desktop and mobile applications in your favorite language without prior experience in frontend development.

You build a UI for your program with Flet controls which are based on Flutter by Google. Flet does not just "wrap" Flutter widgets, but adds its own "opinion" by combining smaller widgets, hiding complexities, implementing UI best-practices, applying reasonable defaults - all to ensure your apps look cool and professional without extra efforts.

Flet app example

Tutorials

Python - To-Do app
Python - Calculator app


## Packaging a desktop app​

Flet Python app and all its dependencies can be packaged into an executable and user can run it on their computer without installing a Python interpreter or any modules.

PyInstaller is used to package Flet Python app and all its dependencies into a single package for Windows, macOS and Linux. To create Windows package, PyInstaller must be run on Windows; to build Linux app, it must be run on Linux; and to build macOS app - on macOS.

Start from installing PyInstaller:

```
pip install pyinstaller
```

Navigate to the directory where your .py file is located and build your app with the following command:

```
pyinstaller your_program.py
```

Your bundled Flet app should now be available in dist/your_program folder. Try running the program to see if it works.

On macOS/Linux:

```
./dist/your_program/your_program
```

on Windows:

```
dist\your_program\your_program.exe
```

Now you can just zip the contents of dist/your_program folder and distribute to your users! They don't need Python or Flet installed to run your packaged program - what a great alternative to Electron!

You'll notice though when you run a packaged program from macOS Finder or Windows Explorer a new console window is opened and then a window with app UI on top of it.

You can hide that console window by rebuilding the package with --noconsole switch:

```
pyinstaller your_program.py --noconsole --noconfirm
```

Bundling to one file​
Contents of dist/your_program directory is an app executable plus supporting resources: Python runtime, modules, libraries.

You can package all these in a single executable by using --onefile switch:

```
pyinstaller your_program.py --noconsole --noconfirm --onefile
```

You'll get a larger executable in dist folder. That executable is a self-running archive with your program and runtime resources which gets unpacked into temp directory when run - that's why it takes longer to start "onefile" package.

NOTE
For macOS you can distribute either dist/your_program or dist/your_program.app which is an application bundle.

Customizing package icon​
Default bundle app icon is diskette which might be confusing for younger developers missed those ancient times when floppy disks were used to store computer data.

You can replace the icon with your own by adding --icon argument:

```
pyinstaller your_program.py --noconsole --noconfirm --onefile --icon <your-image.png>
```

PyInstaller will convert provided PNG to a platform specific format (.ico for Windows and .icns for macOS), but you need to install Pillow module for that:

```
pip install pillow
```

Packaging assets​
Your Flet app can include assets. Provided app assets are in assets folder next to your_program.py they can be added to an application package with --add-data argument, on macOS/Linux:

```
pyinstaller your_program.py --noconsole --noconfirm --onefile --add-data "assets:assets"
```

On Windows assets;assets must be delimited with ;:

```
pyinstaller your_program.py --noconsole --noconfirm --onefile --add-data "assets;assets"
```
