# sdb

Record bash_history in a sqlite3 database.

Query or run the commands from the database.

----
The following are shown in the query.
```
+-------+--------+---------------------+------+----------------+----------------+
|  ID   | COUNT  |      DATE/TIME      |STATUS|   DIRECTORY    |▶   COMMAND     |
+-------+--------+---------------------+------+----------------+----------------+
```

----
### Prerequisites

The history time format **MUST** be in epoch.
```
HISTTIMEFORMAT="%s "
```

Only tested on bash version 4 and up.

Tested on GNU tools only.
```
sqlite3

dd

base64

tput
```

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
## Example of sdb commands.

Show commands with a **zero exit status**.
```
sdb -z
```

Show only **10** recent commands.
```
sdb -m 10
```

Show the **exit status** and **directory** of the commands.
```
sdb -ld
```

Search for commands that starts with **zypper** and end with **bash**.
```
sdb -b zypper -e bash
```

Show commands that has been executed **inside** the directory /tmp **and below** it (recursive).
```
sdb -r /tmp
```

Execute the command with the **ID 50**.
```
sdb -E 50
```
Try the help option for more info about how to use sdb.
```
sdb -h
```

----
## Sqlite Commands.
An example of how to query the database.
```
sqlite3 ~/.bash_history.sqlite '.mode column' '.header on' '.width 6 15 7 50 250' 'select id,epoch,exit_status as status,pwd,cmd from history DESC limit 20;' | less -Ss
```

An example of how to update/set/edit the epoch time with the ID that has a value of 129.
```
sqlite3 ~/.bash_history.sqlite "update history set epoch = 1514694219 where id = 129;"
```

An example of how to delete the entry with the ID 7.
```
sqlite3 ~/.bash_history.sqlite "delete from history where id = 7;"
```
----
## Author

* **Jason V. Ferrer** - [Jetchisel](https://github.com/Jetchisel)

----
## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

----
## Acknowledgments

* The autor of dbhist.sh Denis Gladkikh


