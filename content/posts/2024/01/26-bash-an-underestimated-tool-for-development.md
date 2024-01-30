---
title: "Bash: an underestimated tool for development"
date: 2024-01-26T21:51:23+02:00
description: Few words in favor of Bash as a daily tool for automation of processes
tags: ["bash", "best-pratcices", "software-development", "scripting"]
draft: false
---
# Intro

Bash is one of the most traditional tools for scripting on Unix-like systems. Bash scripts can be found in lots of old and large projects,
as well as in system directories. However, I noticed that younger developer teams avoid Bash in automation tooling and prefer other languages,
like Python, Node or whatever interpreter they already have in use to automate their workflow. I absolutely understand this choice: in the first,
it's more comfortable for developers to use a small set of tools which they know well enough.
In the second, let's admit: Bash has weird syntax and it might be annoying to debug the code.
Some insignfficant differences from examples might result in non-operational script, and it could take a while to find out the root cause.
For instance, let me ask: what's the output of this code?

```bash
a="test"

if [[ "$a" == "test" ]]; then
  echo "Ok1"
fi

if [["$a" == "test"]]; then
  echo "Ok2"
fi
```
Is it
```bash
Ok1
Ok2
```
?

But what about:
```bash
Ok1
1.sh: line 9: [[test: command not found
```
?

Yes, the first conditional statement is valid, and the second isn't because of spaces inside square brackets.
Of course, this kind of unexpected (for inexperienced script writer) behavior makes people consider any other
language which is more predictable, like Python. So, why would you ever choose Bash as automation tool?
In this article I will put on my Cap of Bash Advocate and provide some points in favor of Bash.

# Why Bash

## Has all essential features for scripting

Despite being strange, Bash supports everything you need to implement the logic.
Conditions, loops, variable assignments, functions, control operators - everything is included.

## Works out of the box in Unix systems

Are you sure that __[put other language here]__ exists in all systems you are going to run your code on?
There's very low chance that your target system doesn't have Bash.
Usually it takes place inside minimalistic containers which aren't bundled even with core utilities.
But you don't need to worry about Bash on any standard desktop or server distribution.

Of course, you can install __[put other language here again]__, but it will increase the size of your installation.
Instead of having 1KB bash script, you'll spend 1KB __[other language]__ script + several tens or hundreds of megabytes for runtime.

## A standard tool for automation

Whatever you do in Terminal, can be done using Bash. In my opinion, that's a big advantage of command-line interface over UI.
A typical workflow for making a script looks this way - you try some commands in terminal
and then basically just copy them into a file with minor changes.

## The same code for all environments

There might be custom ways to execute commands on remote systems, including cloud providers.
For example, [AWS CodeBuild](https://docs.aws.amazon.com/codebuild/latest/userguide/welcome.html) uses
[buildspec.yaml](https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html) files to describe remote commands for CodeBuild projects.
However, as `buildspec.yaml` is purely AWS feature, you cannot run it locally to troubleshoot any issues.
As a workaround, you can have a Bash script which is called from `buildspec.yaml`. Having this, the script will be runnable in AWS (via buildspec.yaml)
and locally by launching it from terminal.

## Uses the tools installed on your system. Doesn't require libraries

Unlike casual programming languages which use packages to add features to the project, Bash relies on software installed to the system.
For example, to process a JSON string in Python, you probably want to `import json` and call `json.dumps` or `json.loads` functions from there.
However, in Bash you will need a command-line JSON parser tool inslalled to your system, like `jq`.
After installing the tool once, you will be able to use it in any Bash scripts. In fact, there is a benefit of using external tools:
you can rely on application features, such as modal dialogs, prompts, etc instead of coding your way to fetch the user input.

## Stable

Bash is old. In fact, it's older than me. This language is old enough to be extremely stable.
If any script runs now, it will be most likely runnable in the next ten years or more.
My favorite example of instabity is period between ~2005-2015 when the Developer community slowly migrated from Python 2 to Python 3.
In Python 2 `print` used to be a statement and worked like

```python
print "Hello, World!"
```

However, in Python 3 it became a function:

```python
print("Hello, World!")
```

As a result, scripts written for Python 2 didn't worked for Python 3. By saying that Bash is stable,
I mean that you will much less likely have this kind of "surprises" when upgrading to more recent environment.

## Reusable code

Bash doesn't looks as advanced in sense of code re-usability for me, but there is still a couple of ways to re-use code on my mind

### Option 1. Using PATH and reading the standard output

```bash
# File: /usr/local/bin/say_hello.sh

#!/bin/bash

echo "Hello, $1!"


# File: /tmp/test.sh

#!/bin/bash

say_hello.sh John

# In terminal:
/tmp/test.sh    
Hello, John!
```

Of course, the module file might be stored outside of PATH, but in this case you will need to provide a full path to it.
You also can add a directory with your favorite scripts to __PATH__ and invoke them the same way.

### Option 2. Using 'source' command

```bash
# File: /tmp/module.sh

#!/bin/bash

function say_hello() {
  echo "Hello, $1!"
}

# File: /tmp/app.sh

#!/bin/bash

source "module.sh"

say_hello "John"



# In terminal:
bash app.sh
Hello, John!
```

## Handles the standard output and error exit codes

If `-e` flag is set, you don't need to check the exit code of the previous command; the execution will be aborted if any command fails:

```bash
#!/bin/bash

set -e

ls -lah /not/existent/path

echo "Success!"  # <- not gonna happen


# Execution
bash test.sh
ls: cannot access '/not/existent/path': No such file or directory
```

Output of commands (including pipes) could be assigned to a vairable:

```bash
#!/bin/bash

test="$(ls -lah /dev | wc -l)"  # list files in /dev and calculate the number of lines

echo $test


# Execution
bash ./test.sh
229
```

Please note that in other languages you might need to use conditions and extra statements to check the exit code and standard output:

```python
import subprocess

p = subprocess.run("some_command", capture_output=True, text=True)
if p.returncode != 0:
  raise Exception(" Failure!")
output = p.stdout
```

# How it differs from mainstream languages

There are some keypoints that you should keep in mind when using Bash

## Doesn't stop on failure by default

You will need to set
```bash
set -e
```
in beginning of the script to make it stop if any error occurs. Otherwise your script throw errors and keep running the next commands.
This behavior is typically not desired if the next command depend on results of the previous commands.

## Environment variables and usual variables are the same

In other language you usually need a module to read an environment variable. Let's say in NodeJS:

```javascript
// Environment variable: FOO=BAR

let a = 'hello'; // constant
let b = c; // some silly assignment
let c = process.env.FOO; // read an env var using `process` object
```
Basically, a `process` object is a way to access an environment variable. In Python it's an `os.environ` object.

On the contrary, in Bash environment variables are just kind of pre-defined variables:

```bash
# Environment variable: FOO=BAR

A=hello
B=$A
C=$FOO
```

## No arrays, separators only

Bash doesn't have arrays in the way we use them in other languages. 
Instead, Bash uses kind of lists of strings which are separated with space by default, but separator might be re-defined using __IFS__ vairable:

```bash
#!/bin/bash

a="a b c"

for i in $a; do
  echo "TEST A: $i"
done

b="d/e/f"
IFS=/
for i in $b; do
  echo "TEST B: $i"
done

# Output:
bash ./test.sh
TEST A: a
TEST A: b
TEST A: c
TEST B: d
TEST B: e
TEST B: f
```
Instead of array operations, you just need to modify the string (e.g. append items to the end)

## Undeclared variables = empty variables by default

If no `-u` flag is set, you will have empty strings instead of any sort of "undeclared variable" errors:

```bash
#!/bin/bash

echo "Hello, $name!"  # empty string will be here

set -u

echo "Hello, $name!"  # but this statement will fail


# Execution
bash ./test.sh
Hello, !
./test.sh: line 7: name: unbound variable
```

# Some closing words

Every language is good for specific purposes. I am not going to say that Bash should be used wherever it's technically possible,
but it definitely looks good to me for workflow automation. It works well to convert some complex local commands into more simple ones,
in Continuous Integration and in other places where intense usage of Terminal is expected.

# Appendix. What I do using Bash

At the very end of this article I would like to provide some use cases of Bash in my daily life

## Extract secrets from local and cloud storages

There are some secrets (API keys, passwords) used by automation tools which I periodically need to use for troubleshooting.
Some secrets are stored locally (in [pass](https://www.passwordstore.org/)) and remotely (in [AWS SecretsManager](https://aws.amazon.com/secrets-manager/)).
I have a couple of scripts which fetch credentials from both sources.
I'm not going to share the actual code because I need to clean up some stuff from there first, but commands looks approximately like:

```bash
get_my_password.sh /path/to/my/password | pbcopy
```

The command copies the extracted value to clipboard and I never have to view the actual password in order to copy it.
Please note that `pbcopy` is a command to copy a password in Mac OS X. In Linux you most likely need something like `xclip`.

## Make queries to PostgreSQL database

Okay, the previous command extracts a password and copies it to clipboard, but the same script might be used to connect to database.
I have a Kubernetes cluster with several Postgres services installed. To access to some specific service, I type

```bash
kubernetes_connect_pg.sh k8s_namespace_here service_name_here
```

The bash script looks up the `POSTGRES_*` environment variables from the service deployment, reads credentials and signs me into container (i.e. runs `psql`).
Again, I don't need to find and copy the password myself. In addition, the `service_name_here` doesn't have to be an exact value - it's fine to type just a prefix.

## Establish a VPN connection

The usual way to connect to VPN is Cisco AnyConnect, but I can use OpenConnect as an option.
One of my Bash script starts a `tmux` session in the background and runs OpenConnect there. This is how I connect to VPN without any single click in UI.

## Sign in to command-line tools in the morning

My typical working day starts with signing into some set of services.
I can do it during the day on demand, but it's annoying to get a login page when you are in hurry to look up some data.
That's why I prefer to login wherever is possible in the morning. For this purpose I have a script which connects me to VPN,
opens a browser with [SSO page](https://en.wikipedia.org/wiki/Single_sign-on) and runs a simple command in AWS to trigger a sign in dialog.


## Run projects in development mode

Typically programs expect some arguments as input to run.
In development mode, you might want to pass the same parameters on every execution of the app.
It might be annoying to pass them every time. Of course, the most recent commands might be recovered from the history, but sometimes commands get lost there.
Using Bash scripts, you can "freeze" the command that you are using for tests:

```bash
#!/bin/bash

/run/some/init/app.bin

export ENV_VAR=value

ENV_VAR2=value2 \
  python /path/to/python/entry/point.py \
    --pre-defined-param=value \
    --pre-defined-param-2=value2
```