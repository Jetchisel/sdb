# sdb

A bash shell function that records bash_history in a sqlite3 database.
Recorded commands can be queried and executed.

----
The following are shown in the query.
```
+------------+--------+---------------------+----------------------+--------+---------+----------------+----------------+
|     ID     | COUNT  |      DATE/TIME      |      USER@HOST       | STATUS |   TTY   |   DIRECTORY    |▶    COMMAND    |
+------------+--------+---------------------+----------------------+--------+---------+----------------+----------------+
```

----
### Prerequisites

Make sure that access to the command history is **ENABLED**.


First check the output of

```
set -o | grep history
```

The output should show **on**

```
history         on
```

If it is not then set it to **on** in your **~/.bashrc** file.

```
set -o history
```

In some cases you need to put a test if it is enabled already, something like.

```
if [[ "$(set -o | awk '$1 == "history" {print $2}')" = off ]]; then
  set -o history
fi
```

Bash history time format **MUST** be in **epoch**.
```
HISTTIMEFORMAT="%s "
```

Bash version 4+ and the following GNU tools. Most of them are included in the GNU coreutils.

* dd
* rm
* vim
* tput
* date
* uname
* less
* mktemp
* base64
* whoami
* sqlite3
* realpath

----
### Installing

A preferable place would be some directory that is not in your PATH.
like for example **~/dotfiles.** It can be sourced from **~/.bashrc**

Create the directory and cd into it.
```
mkdir -p ~/dotfiles && cd ~/dotfiles
```
Clone the repo from github.
```
git clone https://github.com/Jetchisel/sdb
```
Source sdb.
```
source sdb/sdb
```

Put something like this in your **~/.bashrc**.
```
if [[ -f $HOME/dotfiles/sdb/sdb ]]; then
  source "$HOME/dotfiles/sdb/sdb"
fi
```
so during interactive session sdb is sourced.

----
### How does it work

Everything is in the past tense. Because the commands has already been executed.
The output of history is being parsed, **history 2** to be more exact/precise.
**history 1** for most of the commands and **history 2** for some commands.

Commands such as **exec bash** or **source ~/.bashrc** and the likes cannot be
captured with just **history 1** because all of the shellrc files is being
re-read/sourced.

With the limited info that is being shown in the **history 2** command, there is
not much to go on, so we resort to some work arounds and hacks just to fill in
some info in the sqlite3 tables (history and directories).

----
### Example of sdb commands.

Show all 100 commands including the **previous** and **other** shell session.
```
sdb -a
```
Show commands with a **zero exit status**.
```
sdb -n 0
```

Show only **10** recent commands.
```
sdb -m 10
```

Show the **exit status** and **directory** of the commands.
```
sdb -ld
```

Search for commands that starts with **zypper** and ends with **bash**.
```
sdb -b zypper -e bash
```

Show commands that has been executed **inside** the directory /tmp **and below** it (recursive).
```
sdb -r /tmp
```

Execute the command with the **ID 50**. Commands executed using the **EVAL** builtin command from bash.
```
sdb -E 50
```

Search commands from the **last ten years** and **now**, following the **GNU date** words syntax separated by a comma.
```
sdb -cam+ -t '-10 years, now'
 ```

Try the help option for more info about how to use sdb.
```
sdb --help
```

---
### Multi user
For multi user option ala central database for users some changes needs to be done.

First create a directory where the shared history database will reside as opposed to the default location.
```
mkdir /path/to/new/directory
```

Put the line below in your ~/.bashrc,  if you're sourcing sdb from ~/.bashrc preferably before that line.
```
SDB_DATABASE=/path/to/new/directory/bash_history.sqlite
```
or whatever you want to name that database instead of **bash_history.sqlite**

Users that will use the database **MUST** have **read** and **write** access to the **directory** and the **database**.
Assuming that the default group is being used. (Otherwise create a new group)
```
chmod -Rv g+rw /path/to/new/directory/
 ```

**DO NOT** use the default location of the database for multi user set up!

If you are more adventurous you can add the root user (not recommended but doable).

Try a network share for the database as well :).

----
### Sqlite Commands.
An example of how to query the database assuming the default name and location from the script.
```
sqlite3 ~/.bash_history.sqlite '.mode column' '.header on' '.width 6 15 7 9 20 50 50' 'select id,epoch,exit_status as status,tty,user_hosts,pwd,cmd from history DESC limit 20;' | less -Ss
```

An example of how to update/set/edit the epoch time with the ID that has a value of 129.
```
sqlite3 ~/.bash_history.sqlite "update history set epoch = 1514694219 where id = 129;"
```

An example of how to delete the entry with the ID 7.
```
sqlite3 ~/.bash_history.sqlite "delete from history where id = 7;"
```

An example of how to **delete** **ALL** commands that matches **sdb** or **'sdb '**

```
sqlite3 ~/.bash_history.sqlite "delete from history where cmd = 'sdb' or cmd like 'sdb %';"
```

---
### Security

If security is an issue then you are at the wrong place!

This is for recording bash history for a user or a group of trusted users.

If the builtin command **eval** is not for you, just ignore the **-E** option, choice number **4**.

but... but... sql injection...

---
### Trouble shooting

Error where it says something about **could not write to database**

  *  Check the permission of the **directory** and the database **file**

The value of the **COMMAND** column is **HISTTIMEFORMAT="%s "**

  *  Set the time format **literally** to **HISTTIMEFORMAT="%s "** in your **~/.bashrc** file

Error where it says database or table is missing.

  *  Check the database if it is still in place.

  *  Check if the table has been removed.

The command **exit** does not exit the current shell session.

  *  Invoke the builtin by:  **builtin exit**

  *  Set the time format **literally** to **HISTTIMEFORMAT="%s "** in your **~/.bashrc** file

The exit_status column  value is **777**

  *  Check if the first value of **"$PROMPT_COMMAND"** is still **__sdb_prompt**

  *  Check for the string **Last_Command** in the sdb script.  It should be just below the
     line where the **__sdb_prompt** function is.

To fix the column  Replace the 777 exit status with something else. (Assuming the default location of the database)

```
  sqlite3 ~/.bash_history.sqlite "UPDATE history SET exit_status = 'something' where id = 'anotherthing';"
```

Change the value of **something** which is the **777** and **anotherthing** which is the **ID** or the first column into a digit.

----
### Miscellaneous

Add a new function that can navigate through the previous directories by parsing the directories table created by sdb.

* **sdc** (https://github.com/Jetchisel/sdc)
----
## Author

* **Jason V. Ferrer** - [Jetchisel](https://github.com/Jetchisel)

----
## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

----
## Acknowledgments

* The author of dbhist.sh Denis Gladkikh, - [Denis](https://www.outcoldman.com/en/archive/2017/07/19/dbhist)
* D.J. Mills, - [e36freak](https://github.com/e36freak)
* Others that was not mentioned :)
