https://flet.dev/docs/

https://github.com/flet-dev/flet


- [Introduction](#introduction)
  - [app example](#flet-app-example)
  - [Controls reference](#controls-reference)
- [Packaging a desktop app​](#packaging-a-desktop-app​)
- [Deploying a web app](#deploying-a-web-app​)
  - [flyio](#flyio​)
  - [replit](#replit​)

## Introduction

What is Flet
Flet is a framework that allows building interactive multi-user web, desktop and mobile applications in your favorite language without prior experience in frontend development.

You build a UI for your program with Flet controls which are based on Flutter by Google. Flet does not just "wrap" Flutter widgets, but adds its own "opinion" by combining smaller widgets, hiding complexities, implementing UI best-practices, applying reasonable defaults - all to ensure your apps look cool and professional without extra efforts.

### Flet app example

Tutorials

Python - To-Do app
Python - Calculator app

### Controls reference

https://flet.dev/docs/controls

Flet UI is built of controls. Controls are organized into hierarchy, or a tree, where each control has a parent (except Page) and container controls like Column, Dropdown can contain child controls, for example:

```
Page
 ├─ TextField
 ├─ Dropdown
 │   ├─ Option
 │   └─ Option
 └─ Row
     ├─ ElevatedButton
     └─ ElevatedButton
```

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

## Deploying a web app​

Flet app can be deployed as a "standalone" web app which means both your Python app and Flet web server are deployed together as a bundle.

Flet apps use WebSockets for real-time partial updates of their UI and sending events back to your program. When choosing a hosting provider for your Flet app you should pay attention to their support of WebSockets. Sometimes WebSockets are not allowed or come as a part of more expensive offering, sometimes there is a proxy that periodically breaks WebSocket connection by a timeout (Flet implements re-connection logic, but it could be unpleasant behavior for users of your app anyway).

Another important factor while choosing a hosting provider for Flet app is latency. Every user action on UI sends a message to Flet app and the app sends updated UI back to user. Make sure your hosting provider has multiple data centers, so you can run your app closer to the majority of your users.

NOTE
We are not affiliated with hosting providers in this section - we just use their service and love it.

### Fly.io​

Fly.io has robust WebSocket support and can deploy your app to a data center that is close to your users. They have very attractive pricing with a generous free tier which allows you to host up to 3 applications for free.

To get started with Fly install flyctl and then authenticate:

flyctl auth login

To deploy the app with you have to add the following 3 files into the folder with your Python app.flyctl

Create with a list of application dependencies. At minimum it should contain module:requirements.txtflet

requirements.txt

```
flet>=0.1.33
```

Create describing Fly application:fly.toml

fly.toml
```
app = "<your-app-name>"

kill_signal = "SIGINT"
kill_timeout = 5
processes = []

[env]
  FLET_SERVER_PORT = "8080"

[experimental]
  allowed_public_ports = []
  auto_rollback = true

[[services]]
  http_checks = []
  internal_port = 8080
  processes = ["app"]
  protocol = "tcp"
  script_checks = []

  [services.concurrency]
    hard_limit = 25
    soft_limit = 20
    type = "connections"

  [[services.ports]]
    force_https = true
    handlers = ["http"]
    port = 80

  [[services.ports]]
    handlers = ["tls", "http"]
    port = 443

  [[services.tcp_checks]]
    grace_period = "1s"
    interval = "15s"
    restart_limit = 0
    timeout = "2s"
```

Replace with desired application name which will be also used in application URL, such as .<your-app-name>https://<your-app-name>.fly.dev

Note we are setting the value of environment variable to which is an internal TCP port Flet web app is going to run on.FLET_SERVER_PORT8080

Create containing the commands to build your application container:Dockerfile

Dockerfile
```
FROM python:3-alpine

WORKDIR /app

COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8080

CMD ["python", "./main.py"]
```
main.py is a file with your Python program.

NOTE
Fly.io deploys every app as a Docker container, but a great thing about Fly is that it provides a free remote Docker builder, so you don't need Docker installed on your machine.

Next, switch command line to a folder with your app and run the following command to create and initialize a new Fly app:

```
flyctl apps create --name <your-app-name>
```

Deploy the app by running:

```
flyctl deploy
```

That's it! Open your app in the browser by running:

```
flyctl apps open
```

### Replit​
Replit is an online IDE and hosting platform for web apps written in any language. Their free tier allows running any number of apps with some performance limitations.

To run your app on Replit:

Sign up on Replit.
Click "New repl" button.
Select "Python" language from a list and provide repl name, e.g. .my-app
Click "Packages" tab and search for package; select its latest version.flet
Click "Secrets" tab and add variable with value .FLET_SERVER_PORT5000
Switch back to "Files" tab and copy-paste your app into .main.py
Run the app. Enjoy.