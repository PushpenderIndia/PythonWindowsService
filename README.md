# Steps To Create an Python Windows Service
 
## (1) Copy Paste These Codes to a Python File (e.g. server.py)
```
import servicemanager
import sys
import win32serviceutil
from mainserver import FlaskServer   # Import your code, I've written a module called mainserver which contains FlaskServer Code using OOPs
import threading
import concurrent.futures
import time

class workingthread(threading.Thread):
    def __init__(self, quitEvent):
        self.quitEvent = quitEvent
        self.waitTime = 1
        threading.Thread.__init__(self)

    def run(self):
        try:
            # Running start_flask() function on different thread, so that it doesn't blocks the code
            executor = concurrent.futures.ThreadPoolExecutor(max_workers=5)
            executor.submit(self.start_flask)
        except:
            pass

        # Following Lines are written so that, the program doesn't get quit
        # Will Run a Endless While Loop till Stop signal is not received from Windows Service API
        while not self.quitEvent.isSet():  # If stop signal is triggered, exit
            time.sleep(1)

    def start_flask(self):
        # This Function contains the actual logic, of windows service
        # This is case, we are running our flaskserver
        test = FlaskServer()
        test.start()

class FlaskService(win32serviceutil.ServiceFramework):
    _svc_name_ = "AA Testing"
    _svc_display_name_ = "AAA Testing"
    _svc_description_ = "This is my service"

    def __init__(self, args):
        win32serviceutil.ServiceFramework.__init__(self, args)
        self.hWaitStop = threading.Event()
        self.thread = workingthread(self.hWaitStop)

    def SvcStop(self):
        self.hWaitStop.set()

    def SvcDoRun(self):
        self.thread.start()
        self.hWaitStop.wait()
        self.thread.join()


if __name__ == '__main__':
    if len(sys.argv) == 1:
        servicemanager.Initialize()
        servicemanager.PrepareToHostSingle(FlaskService)
        servicemanager.StartServiceCtrlDispatcher()
    else:
        win32serviceutil.HandleCommandLine(FlaskService)
```

## (2) Install the latest pywin32.exe 

- NOTE: If you install pywin32 using pip, then it will not going to install properly, thus will show you some errors,
- Visit https://github.com/mhammond/pywin32/releases
- And Download the latest exe, If you are using Python 32bit then download `pywin32-302.win32-py3.x.exe`
- If using Python 64 bit, then download `pywin32-302.win-amd64-py3.x.exe`

## (3) Compile your server.py using pyinstaller

* Compiling service executable
```
C:\Users\Pushpender\Desktop> python -m pip install servicemanager
C:\Users\Pushpender\Desktop> pyinstaller --onefile server.py --hidden-import=win32timezone --clean --uac-admin
```

* Installing & Running service executable  (Run CMD as Administrator)
```
C:\WINDOWS\system32>cd C:\Users\Pushpender\Desktop>
C:\WINDOWS\system32>d:
C:\Users\Pushpender\Desktop>server.exe --startup=auto install       # Installing service with startup == Automatic    
C:\Users\Pushpender\Desktop>server.exe start    # For starting service (You can start from Windows Service or From Task Manager)
C:\Users\Pushpender\Desktop>server.exe stop     # For stopping service (You can stop from Windows Service or From Task Manager)
C:\Users\Pushpender\Desktop>server.exe remove   # For removing installed service
```

## (4) You can run server.py directly without compiling (Run CMD as Administrator)
```
C:\WINDOWS\system32>cd C:\Users\Pushpender\Desktop>
C:\WINDOWS\system32>d:
C:\Users\Pushpender\Desktop>python server.py --startup=auto install   # Installing service with startup == Automatic   
C:\Users\Pushpender\Desktop>python server.py start     # For starting service (You can start from Windows Service or From Task Manager)
C:\Users\Pushpender\Desktop>python server.py stop      # For stopping service (You can stop from Windows Service or From Task Manager)
C:\Users\Pushpender\Desktop>python server.py remove    # For removing installed service
```

## NOTE:
* You can tweak the above code, for example, you can change the following things
  ```_svc_name_ = "AA Testing"
    _svc_display_name_ = "AAA Testing"
    _svc_description_ = "This is my service"
  ```

* Also you can change the classes names of `FlaskService` , `workingthread`
* Please change this line in your code `from mainserver import FlaskServer`
* You can change the startup type to lots of things, type `server.exe --help` in order to know more about settings
* If you want to set the StartUp= Manaull, then don't use `--startup=auto`, while installing service
* If you want to set the StartUp= Automatic (Delayed), then use `--startup=delayed`, while installing service
* Use `--startup` argument before `install` argument
* If you have used any kind of paths in your code such as "/logging/logs.txt", then Make sure to use full/absolute paths in your code, e.g "C:/logging/logs.txt". Because windows will going to call your service from any other path
* Use this command in order to run your service in debug mode: `server.exe debug`

## (4) Integrate Windows server with Inno Setup Builder
* If you want to create a Installer which will Install your service at Installion process & will remove it at Uninstall, then
* Add these block of code in your `script.iss`
```
[Run]
Filename: "{app}\{#MyAppExeName}"; StatusMsg: "Installing Windows Service ... "; Parameters: "--startup=delayed install";  Flags: runhidden waituntilterminated  
Filename: "{app}\{#MyAppExeName}"; StatusMsg: "Running Windows Service ... "; Parameters: "start";  Flags: runhidden waituntilterminated

[UninstallRun]
Filename: "{app}\{#MyAppExeName}"; Parameters: "remove";  Flags: runhidden
```


