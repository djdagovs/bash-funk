# Bash-Funk "processes" module

[//]: # (THIS FILE IS GENERATED BY BASH-FUNK GENERATOR)

The following commands are available when this module is loaded:

1. [-get-child-pids](#-get-child-pids)
1. [-get-parent-pid](#-get-parent-pid)
1. [-get-toplevel-parent-pid](#-get-toplevel-parent-pid)
1. [-kill-childs](#-kill-childs)
1. [-test-processes](#-test-processes)
1. [License](#license)


## <a name="-get-child-pids"></a>-get-child-pids

```
Usage: -get-child-pids [OPTION]... [PARENT_PID]

Recursively prints all child PIDs of the process with the given PID.

Parameters:
  PARENT_PID 
      The process ID of the parent process. If not specified the PID of the current Bash process is used.

Options:
    --help 
        Prints this help.
    --printPPID 
        Specifies to also print the PID of the parent process.
    --selftest 
        Performs a self-test.
```

*Implementation:*
```bash
if [[ ! $_PARENT_PID ]]; then _PARENT_PID=$$; fi
local CHILD_PIDS # intentional declaration in a separate line, see http://stackoverflow.com/a/42854176
childPids=$(command ps -o pid --no-headers --ppid $_PARENT_PID 2>/dev/null | sed -e 's!\s!!g'; exit ${PIPESTATUS[0]})
if [[ $? != 0 ]]; then
    echo "No process with PID ${1} found"'!'
    return 1
fi
for childPid in $childPids; do
    ${FUNCNAME[0]} --printPPID $childPid
done
if [[ $_printPPID ]]; then
    echo $_PARENT_PID
fi
```


## <a name="-get-parent-pid"></a>-get-parent-pid

```
Usage: -get-parent-pid [OPTION]... [CHILD_PID]

Prints the PID of the parent process of the child process with the given PID.

Parameters:
  CHILD_PID 
      The process ID of the child process. If not specified the PID of the current Bash process is used.

Options:
    --help 
        Prints this help.
    --selftest 
        Performs a self-test.
```

*Implementation:*
```bash
if [[ ! $_CHILD_PID ]]; then _CHILD_PID=$$; fi
local parentPid # intentional declaration in a separate line, see http://stackoverflow.com/a/42854176
parentPid=$(cat /proc/${_CHILD_PID}/stat 2>/dev/null | awk '{print $4}'; exit ${PIPESTATUS[0]})
if [[ $? != 0 ]]; then
    echo "No process with PID ${_CHILD_PID} found"'!'
    return 1
fi
echo $parentPid
```


## <a name="-get-toplevel-parent-pid"></a>-get-toplevel-parent-pid

```
Usage: -get-toplevel-parent-pid [OPTION]... [CHILD_PID]

Prints the PID of the top-level parent process of the child process with the given PID.

Parameters:
  CHILD_PID 
      The process ID of the child process. If not specified the PID of the current Bash process is used.

Options:
    --help 
        Prints this help.
    --selftest 
        Performs a self-test.
```

*Implementation:*
```bash
if [[ $_CHILD_PID ]]; then _CHILD_PID=$$; fi
local pid=$_CHILD_PID
while [[ $pid != 0 ]]; do
    pid=$(${BASH_FUNK_PREFIX}-get-parent-pid ${pid})
    if [[ $? != 0 ]]; then
        echo $pid
        return 1
    fi
done
echo ${pid}
```


## <a name="-kill-childs"></a>-kill-childs

```
Usage: -kill-childs [OPTION]... SIGNAL [PARENT_PID]

Sends the given kill signal to all child processes of the process with the given PID.

Parameters:
  SIGNAL (required)
      The kill signal to be send, eg. 9=KILL or 15=TERM.
  PARENT_PID 
      The process ID of the parent process. If not specified the PID of the current bash process is used.

Options:
    --help 
        Prints this help.
    --selftest 
        Performs a self-test.
```

*Implementation:*
```bash
if [[ ! $_PARENT_PID ]]; then _PARENT_PID=$$; fi
local childPids=$(${BASH_FUNK_PREFIX}-get-child-pids $_PARENT_PID)
if [[ $? != 0 ]]; then
    echo $childPids
    return 1
fi
for childPid in $childPids; do
    echo "Killing process with PID $childPid..."
    kill -s $_SIGNAL $childPid 2> /dev/null || :
done
```


## <a name="-test-processes"></a>-test-processes

```
Usage: -test-processes [OPTION]...

Performs a selftest of all functions of this module by executing each function with option '--selftest'.

Options:
    --help 
        Prints this help.
    --selftest 
        Performs a self-test.
```

*Implementation:*
```bash
${BASH_FUNK_PREFIX:-}-get-child-pids --selftest && echo || return 1
${BASH_FUNK_PREFIX:-}-get-parent-pid --selftest && echo || return 1
${BASH_FUNK_PREFIX:-}-get-toplevel-parent-pid --selftest && echo || return 1
${BASH_FUNK_PREFIX:-}-kill-childs --selftest && echo || return 1
```


## <a name="license"></a>License

Copyright (c) 2015-2017 Vegard IT GmbH, http://vegardit.com

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
