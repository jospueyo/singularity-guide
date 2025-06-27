# Tips to use the cluster

## Create aliases to make your life easier

Aliases are commands that are shortcuts to other longer commands.
For instance, an alias can be created to enter ICRA's cluster. Instead of typing:

```bash
ssh <username>@<ipadress>
```
You can create an alias that do the same by typing:

```bash
clustericra
```

To create an alias, the `.bashrc` file must be edited. Type:

```bash
cd
nano .bashrc
```
And add the following line:

```bash
alias clustericra="ssh <username>@<ipadress>"
```
Of course, you should replace `<username>` by your username and `<ipadress>` by the IP adress of the cluster.

You can also create aliases to execute programs inside your singularity containers. For instance, adding the following line executes `runswmm` which is installed in a container called `container-swmm`:

```bash
alias runswmm="singularity exec /home/jpueyo/container-swmm /opt/Stormwater-Management-Model/build/bin/runswmm"
```

## TMUX

**tmux** (terminal multiplexer) is a powerful command-line tool that allows users to manage multiple terminal sessions within a single window. It enables multitasking and better organization by letting you split windows, detach and reattach sessions, and persist workflows even if you lose SSH connectivity. 

In plain words, this means that you don't need to keep your laptop on and connected to the cluster to run your programs.

### Basic Concepts

- **Session**: A collection of windows.
- **Window**: A single view containing one or more panes.
- **Pane**: A subdivided section of a window, acting like a separate terminal.

### Using tmux

Start with a named session:

`tmux new -s session_name`

Attach to an existing session:

`tmux attach -t session_name`

Attach to the last session:

`tmux a`

List all sessions:

`tmux ls`

Detach from a session:

`Ctrl-b d`

To execute any order in tmux using key bindings, first type Ctrl+b and then the key corresponding to that order. There is only one exception: type Ctrl+d to close a pane (no program must be running to be able to close the pane).

Below there are the most used key bindings:

| Action                  | Key Binding            |
| ----------------------- | ---------------------- |
| Split pane vertically   | `Ctrl-b "`             |
| Split pane horizontally | `Ctrl-b %`             |
| Change active pane      | `Ctrl-b o`             |
| Start new window        | `Ctrl-b c`             |
| Switch to next window   | `Ctrl-b n`         |
| Switch window           | `Ctrl-b [0-9]`         |
| Rename current window   | `Ctrl-b ,`             |
| Move between panes      | `Ctrl-b [arrow key]`   |
| Resize pane             | `Ctrl-b :resize-pane`  |
| Close current pane      | `Ctrl-b x`             |
| Scroll through history  | `Ctrl-b [`             |
| Detach session          | `Ctrl-b d`             |
| Kill session            | `Ctrl-b :kill-session` |
| List all key bindings   | `Ctrl-b ?`             |


## Move between nodes and choose one

When you enter the cluster you are in the central computer `undarius`, to run your computations you should use the computational nodes. You can access them using `ssh compute-<X>`, replacing `<X>` by the number of the node. Once in the node, you can run `htop` to visualize if other user is using the node and choose a node that is available.

## Run commands in parallel

The easiest way to run commands in parallel is creating multiple panes in tmux and executing one command in each pane. However, to automate this process, a bash script can be used. The basic structure to run commands in parallel is:

```bash
parallel <command> ::: <args>
```

For instance:

```console
parallel Rscript ::: script1 script2 script3 script4
```
You can also use a text file with a list of arguments:

```txt
# args.txt
script1
script2
script3
```

```bash
parallel -a args.txt Rscript
```

Both options will run in parallel the 4 scripts used as arguments to `Rscript`.

In case you have a command that requires more than one argument, you can pass them using the console:

```console
parallel runswmm ::: 1.inp 2.inp 3.inp :::+ 1.rpt 2.rpt 3.rpt :::+ 1.out 2.out 3.out
```

Or using a txt file:

```txt
# args.txt
1.inp 1.rpt 1.out
2.inp 2.rpt 2.out
3.inp 3.rpt 3.out
```

```console
parallel -a test_args.txt --colsep ' ' ./test.sh
```
Both options above will run three SWMM simulations using the 3 INP files specified and saving the reports and outputs in the subsequent files.

If your arguments have blank spaces (they shouldn't to avoid further problems), you can use any special character as separator:

```txt
# args.txt
prova 1.inp|prova 1.rpt|prova 1.out
prova 2.inp|prova 2.rpt|prova 2.out
prova 3.inp|prova 3.rpt|prova 3.out
```

```console
parallel -a args.txt --colsep '\|' runswmm
```
Notice the symbol `\` before `|`, this is necessary to escape the character.







