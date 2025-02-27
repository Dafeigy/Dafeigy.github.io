---
title: Street Fighter vs. DQN Fighter
mathjax: true
tags:
- 强化学习
- 街头霸王
- DQN
categories:
- 强化学习
- 游戏智能
---



## Introduction

I would like to apply an improved DQN to the Game Street Fighter V, an action game published by CAPCOM in 2016. This project was inspired by LinYi and I try to use DQN without HPE(Human Pose Estimation) due to the limitation of my hardware.

<img src="https://s2.loli.net/2022/05/27/DSw9adIUvW3q86K.png" style="zoom:67%;" />

I try to train a model with less time training and more robust to any other Acion fighting game. This may be cool and difficult since I need to prepare my exam while handling other projects in this very year. 

<!--more-->

## Preparing

### Before we start

We need access to some in-game factors that help game agent better comprehend the situation(Though these factors are simply shown in the screen,I would like to use in-game memory to get which is faster and relieve GPU's burden if use CV method to convert these images' info).



<img src="https://i.loli.net/2021/09/08/84P5qBKMUWn6JLb.png" alt="image-20210908230408019" style="zoom: 67%;" />

However,CAPCOM use anti-spam to block the access to read memory while the game runs,which may crash the game if you turn some basic memory search tools like `CheatEngine`.I use this [Tool](https://bbs.pediy.com/thread-195729.htm) to avoid detection and it actually works!(By the way ,the unzip code is `qq295991`).

In addition,you should open the software before you launch the game,and change the default settings to this:

<img src="https://i.loli.net/2021/09/08/sGI6nJjkTADOZbd.png" alt="image-20210908230121851" style="zoom: 80%;" />



Then you can search the factors you need and locate their base&offset address.

<img src="https://i.loli.net/2021/09/08/13NdY7S6WtCLczw.png" alt="image-20210908230121851" style="zoom: 80%;" />



The Following Problem is that when I use `openProcess` in `win32api` to get value of the factors it returns the error like this:

```python
Traceback (most recent call last):
  File "G:/SFV/Tool/factors.py", line 180, in <module>
    hp = Gamefactors()
  File "G:/SFV/Tool/factors.py", line 55, in __init__
    hProcess = win32api.OpenProcess(
pywintypes.error: (5, 'OpenProcess', 'Access Denied.')
```

It really takes me a long time to find the solution:( I look up the MSDN and find this:

> If the caller has enabled the SeDebugPrivilege privilege, the requested access is granted regardless of the contents of the security descriptor.

Which means I need the `SeDebugPrivilege` huh...So I write a function to get more privileges:

```python
def get_extra_privs():  # get more access to the game
    # Try to give ourselves some extra privs (only works if we're admin):
    # SeBackupPrivilege   - so we can read anything
    # SeDebugPrivilege    - so we can find out about other processes (otherwise OpenProcess will fail for some)
    # SeSecurityPrivilege - ??? what does this do?

    th = win32security.OpenProcessToken(win32api.GetCurrentProcess(),
                                        win32con.TOKEN_ADJUST_PRIVILEGES | win32con.TOKEN_QUERY)
    privs = win32security.GetTokenInformation(th, win32security.TokenPrivileges)
    newprivs = []
    for privtuple in privs:
        if privtuple[0] == win32security.LookupPrivilegeValue(None, "SeBackupPrivilege") or privtuple[
            0] == win32security.LookupPrivilegeValue(None, "SeDebugPrivilege") or privtuple[
            0] == win32security.LookupPrivilegeValue(None, "SeSecurityPrivilege"):
            # print("Added privilege " + str(privtuple[0]))
            newprivs.append((privtuple[0], 2))  # SE_PRIVILEGE_ENABLED
        else:
            newprivs.append((privtuple[0], privtuple[1]))

    # Adjust privs
    privs = tuple(newprivs)
    win32security.AdjustTokenPrivileges(th, False, privs)
```

 Now we need to access these factors: 

* Player&Rival's HP
* Player&Rival's EX
* Player&Rival's VT
* Player&Rival's location

Yet we need to notice that player's & rival's location are not in float type while HPs are in normal byte type, so we need a function to transfer Byte type to Float type:

```python
def byte2Float(s):  # Convert byte to float
    try:
        i = int(s, 10)  # convert from Dec to a Python int
        cp = pointer(c_int(i))  # make this into a c integer
        fp = cast(cp, POINTER(c_float))  # cast the int pointer to a float pointer
        a = fp.contents.value
    except:
        a = 0
    finally:
        return a
```

### Why not define a class?

We can add function to this class to reduce the time spent on accessing these factors:

```python
class Hp_getter():
    def __init__(self):
        get_extra_privs()
        hd = win32gui.FindWindow(None, "StreetFighterV")
        pid = win32process.GetWindowThreadProcessId(hd)[1]
        self.process_handle = win32api.OpenProcess(0x1F0FFF, False, pid)
        self.kernal32 = ctypes.windll.LoadLibrary(r"C:\\Windows\\System32\\kernel32.dll")

        self.hx = 0
        # get dll address
        hProcess = Kernel32.OpenProcess(
            PROCESS_QUERY_INFORMATION | PROCESS_VM_READ,
            False, pid)
        hModule = EnumProcessModulesEx(hProcess)
        for i in hModule:
            temp = win32process.GetModuleFileNameEx(self.process_handle, i.value)
            if temp[-18:] == "StreetFighterV.exe":
                self.StreetFighter=i.value
                print(self.StreetFighter)
```

And by the way no thanks:

```python
def EnumProcessModulesEx(hProcess):
    buf_count = 256
    while True:
        LIST_MODULES_ALL = 0x03
        buf = (ctypes.wintypes.HMODULE * buf_count)()
        buf_size = ctypes.sizeof(buf)
        needed = ctypes.wintypes.DWORD()
        if not Psapi.EnumProcessModulesEx(hProcess, ctypes.byref(buf), buf_size, ctypes.byref(needed),
                                          LIST_MODULES_ALL):
            raise OSError('EnumProcessModulesEx failed')
        if buf_size < needed.value:
            buf_count = needed.value // (buf_size // buf_count)
            continue
        count = needed.value // (buf_size // buf_count)
        return map(ctypes.wintypes.HMODULE, buf[:count])
```











