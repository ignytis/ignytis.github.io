---
title: "Crafting a good command-line application"
date: 2023-01-06T09:45:02+02:00
draft: false
---
# Intro
Command-line tools take a special place in IT world in sense of target audience. CLI is not that kind of apps that you deliver to casual end-user. On the contrary, they are mostly used by developers, system administrators, backend services, and red-eyed Unix enthusiasts. Although direct impact of CLI tool to business is most likely small (or none at all), the tool still needs to be good enough to use. If application is not convenient enough, users will waste a lot of time on configuration or figuring out how does it works, and might end up with looking for some alternative solution. In this article I would like to share my ideas about handy features which an CLI-application could have.

# Use stdio, but have arguments as a fallback
As for me, pipeline is one of the most amazing features in Unix-like systems. Thanks to pipelines an input of one application might be processed by another one. To apply some advanced logic for data processing, a set of applications could be chained using Unix pipe. If your application accepts a single block of data as input (e.g. text, JSON document or binary data), it would make sense to read it via standard input. It gives user an opportunity to pass the input in different ways:

```bash
$ cat /path/to/file | my_app # pass contents of a file, option 1
# or
$ my_app < /path/to/file # pass contents of a file, option 2
# or
$ echo "some content" | my_app # whatever input
# or
$ myapp << EOF # might be useful in scripts to hard-code the file contents
some
multi-line
input
EOF
```
The same with output: if the app produces only a single piece of text/binary data, it might be written to standard output. In this case user would be able to:

```bash
$ my_app > /path/to/file.txt # write it to a file if needed
$ my_app >> /path/to/file.txt # ...or append it to an existing file
$ my_app # print it to terminal if no logging is needed
$ my_app | gzip > /path/to/file.txt.gz # process the output of app
```
Unfortunately, it’s not guaranteed that standard input and output are available for use in all environments. For example, terminal might be not allocated if app is running inside Docker container or via SSH command. To handle such cases, the application might have optional arguments to specify an input and output file, like --input and --output.

# Parseable output
As it was mentioned in the previous section, you might want to use an output of your application in other tools. To achieve this, the output has to be machine-readable, or parseable. If output is a single piece of data, it would be sufficient to print it to stdandard output as is:

```bash
$ ./answer_to_the_ultimate_question_of_life_the_universe_and_everything
42
However, the application might have more than one output or a complex structure as an output:

$ ./complex_number_parser "3+2i"
Real part is: 3
Imaginary part is: 2
```
Are real and imaginary parts easily parseable here? Not really. To read the real and imaginary part, you’ll need, for instance, to take one line from output and split it by space:

```bash
$ ./complex_number_parser "3+2i"| head -n 1 | rev | cut -d' ' -f 1 | rev
3 # real part
$ ./complex_number_parser "3+2i"| tail -n 1 | rev | cut -d' ' -f 1 | rev
2 # imaginary part
To simplify this process, you can add an option to print the output in machine-readable format and use some common processor utility, like jq:

$ ./complex_number_parser "3+2i" --format=json
{"real": 3, "imaginary": 2}
$ ./complex_number_parser "3+2i" --format=json | jq '.real'
3
$ ./complex_number_parser "3+2i" --format=json | jq '.imaginary'
2
```

# Stdout vs stderr
The general rule is: the main result of your process goes to stdout, the additional information goes to stderr. Despite being called “standard error”, it could take care of other kinds of technical information. If your application outputs the data of strict format (JSON, YAML, raw output from other process, etc), you absolutely don’t want to feed additional output (e.g. logs) to other applications via Unix pipe. Sending the additional data to stderr lets you forward it to /dev/null or to a separate file if needed:

$ myapp
LOG [INFO]: application started
{“output”: “some_value”}
LOG [INFO]: application terminated.
$ myapp 2>/dev/null
{“output”: “some_value”}
$ myapp 2>/path/to/log/file.log
{"output": "some_value"}
$ cat /path/to/log/file.log
LOG [INFO]: application started
LOG [INFO]: application terminated.
Unlike the first case, output of the second and third command is a valid JSON which might be forwarded to some other tool, like jq.

# Non-interactive mode
It’s a good practice to ask from user a confirmation for some actions. Typically, a confirmation is needed for irreversible actions (e.g. deletion of a file or data), as user should have a chance to abort an unwanted action which is caused by mistyped command. However, there are few occasions when user might want to run a command instantly:

They are absolutely sure that the command is valid
If the command is executed by automated script which cannot handle input prompts
If any of these options is expected, a quiet or non-interactive mode would be needed. If quiet mode is enabled, the application automatically confirms all actions, does the job, and then exits.

An example of non-interactive execution is an assume-yes flag for apt-get tool in Ubuntu: https://askubuntu.com/a/672893

Short and long versions of arguments
Let’s take a df command as an example:
```bash
$ df -H /
$ df --human-readable /
```
Should both options exist in a command-line application? I would say, yes. Long options are good for less-experienced users, because the meaning of a command is more self-explanatory. In addition, it might be better to remember a word (or a couple of words) than a letter from alphabet. However, it might be too boring for experienced users to type the long command every time. Just imagine that you need to write that --human-readable flag tens times per day. That’s why it’s recommended to have a short version of a command as well.

One of the mostly used naming conventions for commands is: long commands are prefixed with double-dash, use lower-case and a single dash as a word separator (A.K.A. kebab-case). Short commands are prefixed with a single dash and contain one letter. NB: short commands are often case sensitive, i.e. -X and -x might be different commands. An example from real life is the same df command:

```bash
$ df --help
(some other text)
 -t, — type=TYPE limit listing to file systems of type TYPE
 -T, — print-type print file system type
```
I would prefer NOT to use the same letter in upper and lower case if possible to avoid possible confusion for user.

Logs and verbosity for troubleshooting
The topic of verbosity levels is not as trivial as it might seem, and I would like to make a separate article about that. An important point in context of this article is that an application might have different levels of verbosity, such as debug, info, warning, error and critical. There are three most typical ways to define a log level:

Option 1. Via environment variables:

```bash
$ LOG_LEVEL=info myapp
```
This option is typically used by logging libraries, as they could read the option value directly (you don’t need to pass the value into library, because it’s available from standard modules). For example, an env_logger in Rust handles a RUST_LOG environment variable.

Option 2. Via command-line arguments, long notation:
```bash
$ myapp --log-level=info
```
Option 3. Via command-line arguments, short notation:
```bash
$ myapp -vvv
```
It might be a good shortcut for the second option: the more v characters exist in the command — the higher is verbosity level.

# Graceful shutdown vs killing
There are two ways to ask an application to terminate. Let’s call them a soft way and a hard way.

Using soft way, we say to application: “hey, could you please finish whatever you are doing and quit?”. In Unix it is implemented by SIGINT and SIGTERM signals.

SIGINT (interruption) is usually sent by pressing <CTRL>+C combination and terminates the application by default. However, the application might have a handler which does something else instead of termination. For instance, if you press a <CTRL>+C in Vim editor, you’ll get a message:

Type :qa and press <Enter> to exit Vim

which means that Vim ignores the SIGINT on purpose.

SIGTERM is typically sent by other process. For example, this is how the docker stop command works, or how Kubernetes scales down pods.

Using hard way, we say to application, well, nothing. We just kill it by SIGKILL signal. No handler could stop it, no time is given to the application to finish the job.

Alright, in case of hard way we can do nothing inside application to handle it, but what about SIGINT/SIGTERM? Should your application handle them? I think, it depends on application’s lifetime. For example, a ls command lists a directory and then immediately exists. It has no chance to make user wait until it finishes. It would be challenging to catch a moment and to send a signal to that command. For such short-living commands there is not much point in making a handler. But if it’s expected that application runs for a while (e.g. server, editor, game, converter, downloader, etc), it’s worth some effort to implement a graceful shutdown.

Ideally, graceful shutdown should fit into some short time interval, for example, a minute. If graceful shutdown takes a longer time, user will likely kill the app in a hard way. In some occasions user might want to skip the process of graceful shutdown, and covering such cases will save user’s effort. If user wants to kill an app — they will do it, either using application’s functionality or via process manager. The second way requires a separate terminal and a bit of user’s frustration. To simplify the process, skipping the graceful shutdown might be implemented, for example, by sending a SIGINT twice. In other words, if user types CTRL+C while graceful shutdown is in progress, the app exits immediately.

The data directory
Your application might need some place to store the data. It might be configuration, modules or extensions, game saves, whatever. I’m not mentioning intermediate files and cache here, because it might be placed to /tmp directory which automatically cleans up on each reboot.

A conventional path to store the configuration is ~/.appname, or a directory with your application’s name prefixed with dot and located in your user’s home directory. You can probably see lots of these directories by running ls -lah ~ on your machine. User’s home directory is a good place for application data, as applications run typically on behalf of current user and there will be no problems with reading, writing and cleaning the data in a directory owned by them. By the way, config directory names are often suffixed with .d in Unix systems. For example, a configuration directory for Emacs is ~/.emacs.d. It distinguishes file names from directory names, so you might consider this naming option as well.

In some occasions (e.g. user doesn’t have their own home directory) the configuration directory needs to be located in other path. To override the default option with home directory, an application could use an environment variable, like APPNAME_HOME, so a command to run the app will be e.g.

```bash
$ MYAPP_HOME=/opt/myapp/data myapp
# or
$ export MYPP_HOME=/opt/myapp/data
$ myapp
```
Configuration from file, CLI arguments, environment variables
Your app might have some ~/.yourapp/config.yaml-like file which is loaded by application on each start up. How can we do the configuration more tuneable? Let’s consider the configuration file as a default config storage. It might look like

```yaml
general:
  some_section:
    some_key: some_value
    some_other_key: 1234
  logging:
    level: error
```
What if we want to run an application manually and to have a logging level set to debug instead of error? There are three options:

To make a separate config file. Just kidding. You don’t wanna do it for a single run
Use the environment variables:
$ GENERAL_LOGGING__LEVEL=debug myapp
Use the command line arguments:
$ myapp --config general.logging.level=debug
On application level you might have a module which resolves and merges all components of configuration into final config. Please note that environment variable or parameter names follow the naming in configuration file’s hierarchy; it helps a lot to automate the process of parameter injection using recursive functions.

# Prefixed commands and subcommands
Let’s assume that we are building a CLI-client app for a social network. There is a set of features (posts, friends, profile, private messages, etc) and we would like to group them. There are two approaches to do it:

Option A: a single-level list of commands

```bash
$ mysocial-cli help
posts:add # command 1
posts:list # command 2
posts:delete # ... .. .etc
friends:list
fiends:add
friends:delete
```
In code it most likely looks like appending all commands to the same object:
```python
myapp.add_command(Command(name="posts:add", callback=fn1))
myapp.add_command(Command(name="posts:delete", callback=fn2))
myapp.add_command(Command(name="friends:add", callback=fn3))
# etc
```
Option B: multiple levels aka subcommands
```bash
$ mysocial-cli help
posts
friends
$ mysocial-cli posts help
add
delete
list
```
The code would look like a bit more complex:

```python
subcommand_posts = Subcommand()
subcommand_posts.add_command(Command(name="add", callback=fn1))
subcommand_posts.add_command(Command(name="delete", callback=fn2))
app.add_subcommand(subcommand_posts)

subcommand_friends = Subcommand()
subcommand_friends.add_command(Command(name="add", callback=fn3))
app.add_subcommand(subcommand_friends)
# etc
```
In my opinion, the first option is good for relatively small apps, as it allows to display a complete list of available commands on a single page. However, if application has lots of commands, it might be better to go on with the second option. A good example of a huge application is AWS CLI. This client communicates with tens of AWS components while each component might have tens of commands. User definitely doesn’t want to look at list of hundreds of commands just to find a single one. In addition to readability, the approach with subcommands helps to split the command initialization logic into different modules (files):

```python
# add_command (see a snippet above) calls are encapsulated in 
import PostsSubcommand from mysocial_cli.subcommands.posts
import FriendsSubcommand from mysocial_cli.subcommands.friends

app.add_subcommand(PostsSubcommand())
app.add_subcommand(FriendsSubcommand())
```
# CLI arguments with different data types — repeated, boolean, integer
Technically, all command-line arguments in their raw form are strings. However, for CLI application libraries might convert types of arguments for better user experience. Examples of conversion:
```
--myparam=some_value // string
--myparam=123 // number
--myparam // Boolean (flag; true if specified, false otherwise)
--myparam=some_value --myparam=some_other_value // array of strings
```
Don’t forget to check which options does your CLI library provide. It would save some effort on type casting logic.

# Help output
Online documentation is good, but it is more favorable for user not to switch between terminal and browser to learn the basics of application usage. It is recommended to include at least a brief introduction into help command. The help section should explain the goal of application and list mandatory and optional arguments. It would be nice to provide some examples as well. To check out a good example of help command contents, you can try some standard utility, such as df --help or ls --help. Please keep in mind that you usually DON’T have to format the help section on your own; if your application uses any CLI library (see the last section of this article), is most likely takes care of the HELP command.

help, -help or --help? In my opinion, the option with single dash is the worst one. As it was told in the section about long and short versions of arguments, a single dash is expected from short arguments, while “help” is a long one; -h is fine though. The option with double dash seems to be the mostly used one. Double dash is good for long options, so is “help”. However, in recent days a single “help” with no dashes looks the best option for me. The reason is that dashes are used to mark command options, but is “help” really an option? By calling the help command, user expects to see some documentation and have the program immediately terminated. In this sense “help” looks more like a sub-command which does some job (output) and then finishes. Therefore I would proceed with option with no dashes at all.

# Version info as a command
For troubleshooting purposes it would be good to have a version info in application. Should it be inside help command or a separate version command is more preferable? My vote is for the second option. This gives an advantage for automation; for example, CI/CD needs much less effort to fetch the app version from application output. In addition, it follows the principle of separation of concerns, as version info is not a part of information which helps a user to do something.

Okay, your application prints the version info (e.g. v2.34.32), but is it enough? Is a new version released every time when the application is deployed to test environment? How would you (or your colleagues) know which exactly state of the code runs on the test environment? To shred some light on it, I would add the Git commit hash and the date of building to version info:

```bash
$ myapp version
v12.34.56
f10e2821bbbea527ea02200352313bc059445190
2022–12–03 08:45:33
```

Few decisions to consider:

-version, --version, or version? It’s a separate command, not an option for any other command, so I would go with the third option. See the previous section for details.

stdout or stderr? As the version info is main outcome of the command, it makes sense to print it to stdout.

# Examples of libraries for different languages
## Golang:

Flaggy https://github.com/integrii/flaggy
## PHP:

Symfony Console https://symfony.com/doc/current/components/console.html
## Python:

Argparse (part of Python distribution) https://docs.python.org/3/library/argparse.html
## Rust:

Clap https://docs.rs/clap/latest/clap/

