# Debug

## Description
This package simplifies debug output and makes it more powerful.

## Installation
Run `pip install jc-debug` to install it. This will install the package named "debug" in your site-packages.

## Getting Started
This debug module a class named DebugChannel, instances of which are
useful for adding temporary or conditional debug output to CLI scripts.

The minimal boilerplate is pretty simple:

    from debug import DebugChannel
    dc=DebugChannel(True)

By default, output is sent to stdandard error and formatted as:

```
'{label}: [{pid}] {basename}:{line}:{function}: {indent}{message}\n'
```

There are several variables you can include in DebugChannel's output:

* date: Current date in "%Y-%m-%d" format by default.
* time: Current time in "%H:%M:%S" format by default.
* label: Set in the initializer, defaults to "DC".
* pid: The current numeric process ID.
* pathname: The full path to the source file.
* basename: The filename of the source file.
* function: The name of the function whence dc(...) was called. This will be "__main__" if called from outside any function.
* line: The linenumber whence dc(...) was called.
* code: The text of the line of code that called dc().
* indent: The string (typically spaces) used to indent the message.
* message: The message to be output.

So, for example, if you want to see how your variables are behaving in a
loop, you might do something like this:

```python
from debug import DebugChannel

dc=DebugChannel(
  True,
  fmt="{label}: {line:3}: {indent}{message}\n"
)

dc("Entering loop ...").indent()
for i in range(5):
  dc(f"i={i}").indent()
  for j in range(3):
    dc(f"j={j}")
  dc.undent()("Done with j loop.")
dc.undent()("Done with i loop.")
```

That gives you this necely indented output. The indent() and undent()
methods are one thing that makes DebugChannels so nice to work with.

```
DC:   8: Entering loop ...
DC:  10:   i=0
DC:  12:     j=0
DC:  12:     j=1
DC:  12:     j=2
DC:  13:   Done with j loop.
DC:  10:   i=1
DC:  12:     j=0
DC:  12:     j=1
DC:  12:     j=2
DC:  13:   Done with j loop.
DC:  10:   i=2
DC:  12:     j=0
DC:  12:     j=1
DC:  12:     j=2
DC:  13:   Done with j loop.
DC:  10:   i=3
DC:  12:     j=0
DC:  12:     j=1
DC:  12:     j=2
DC:  13:   Done with j loop.
DC:  10:   i=4
DC:  12:     j=0
DC:  12:     j=1
DC:  12:     j=2
DC:  13:   Done with j loop.
DC:  14: Done with i loop.
```

That's a simple example, but you might be starting to get an idea of
how versatile DebugChannel instances can be.

A DebugChannel can also be used as a function decorator:

```python
import time
from debug import DebugChannel

def delay(**kwargs):
  time.sleep(1)
  return True

dc=DebugChannel(True,callback=delay)
dc.setFormat("{label}: {function}: {indent}{message}\n")

@dc
def example1(msg):
  print(msg)

@dc
def example2(msg,count):
  for i in range(count):
    example1(f"{i+1}: {msg}")

example2("First test",3)
example2("Second test",2)
```

This causes entry into and exit from the decorated function to be
announced in the given DebugChannel's output. If you put that into a
file named foo.py and then run "python3 -m foo", you'll get this:

```
DC: test_basics: Simple test message 2
DC: test_basics: example2('First test',3) ...
DC: example2:     example1('1: First test') ...
1: First test
DC: example2:     example1(...) returns None after 27µs.
DC: example2:     example1('2: First test') ...
2: First test
DC: example2:     example1(...) returns None after 27µs.
DC: example2:     example1('3: First test') ...
3: First test
DC: example2:     example1(...) returns None after 18µs.
DC: test_basics: example2(...) returns None after 647ms.
DC: test_basics: example2('Second test',2) ...
DC: example2:     example1('1: Second test') ...
1: Second test
DC: example2:     example1(...) returns None after 13µs.
DC: example2:     example1('2: Second test') ...
2: Second test
DC: example2:     example1(...) returns None after 20µs.
DC: test_basics: example2(...) returns None after 430ms.
```

That's a very general start.
