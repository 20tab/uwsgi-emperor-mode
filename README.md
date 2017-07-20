# Definition

This is a step-by-step guide to install and configure the uWSGI server in _emperor_ mode and with multiple language support. It is mainly intended for macOS environments but, excluding the parts related to bonjour and `launchd`, it can be adapted to other platforms.

# Configuration

Please make sure to read each of the following sections, and complete them in the specified order.

### uWSGI

uWSGI will be installed from its sources with the support for multiple languages, then a plugin will be built for each language to be supported:
- get uwsgi latest sources from [here](http://uwsgi-docs.readthedocs.io/en/latest/Download.html);
- extract the files in a local directory;
- open a terminal and `cd` into the uwsgi sources directory;
- build the uwsgi binary:
```sh
make PROFILE=nolang
```

- generate the plugins, being sure to already have installed, for each plugin, its corresponding binary (e.g. `python3.5` or `python27`). In the case of various Python versions, execute:
```sh
PYTHON=python2.7 ./uwsgi --build-plugin "plugins/python python27"
PYTHON=python3.6 ./uwsgi --build-plugin "plugins/python python36"
```
- create a local directory where you intend to collect all your uWSGI plugins (e.g. `~/uwsgi/plugins/`);
- copy **all** the plugins to the directory you have just created:
```sh
cp python27_plugin.so ~/uwsgi/plugins/
cp python36_plugin.so ~/uwsgi/plugins/
```
- install the compiled uWSGI binary, e.g.:
```sh
cp uwsgi /usr/local/bin/
```

### Bonjour

This part is only meant for macOS and makes it possible to assign a custom domain to each project (e.g. `my_new_project.local/`):
- clone or download the repository and `cd` in its directory;
- compile the bonjour uWSGI plugin:
```sh
uwsgi --build-plugin uwsgi-bonjour
```
- copy the plugin to the uWSGI plugins directory:
```shell
cp bonjour_plugin.so ~/uwsgi/plugins/
```
Code is taken from Unbit uwsgi-bonjour, for further information check [here](https://github.com/unbit/uwsgi-bonjour).

### Emperor

For more information on uWSGI _emperor_ mode refer to the [official documentation](http://uwsgi-docs.readthedocs.io/en/latest/Emperor.html). Before configuring uWSGI:
- create a directory where to collect all the projects .ini configuration files (e.g. `~/uwsgi/vassals/`);
- create a directory where to collect all the projects log files (e.g. `~/uwsgi/logs/`)

Provided `emperor.ini` contains a sample configuration to run the uWSGI server in _emperor_ mode. Customize it with the paths to your vassals and plugins directories and to the emperor log file, plus any other setting/option you wish to run the emperor with. Then:
- move the customised emperor configuration file to a convenient location:
```shell
cp emperor.ini ~/uwsgi/
```
Be careful not to put the emperor configuration file into the vassals configuration files directory.

### Vassals

The emperor needs to be told the language of each project, therefore make sure to indicate the corresponding plugin in each vassal configuration file:
```INI
plugin = python35
```
##### Bonjour (macOS)

In order to assign a custom domain to a project (e.g. `my_new_project.local/`), also include the following lines in the corresponding vassal configuration file:
```INI
plugin = bonjour
bonjour-register = name=%(project_name).local,cname=localhost
socket = 127.0.0.1:0
subscribe-to = 127.0.0.1:5005:%(project_name).local
```

# Execution

The emperor can now be launched both manually and automatically.

### Manual

This is straightforward, just launch uWSGI as superuser and with the emperor configuration file:

```sh
sudo uwsgi ~/uwsgi/emperor.ini
```

### Automatic (macOS)

Provided `it.unbit.uwsgi.emperor.plist` is a sample property list file that instructs `launchd` to run the emperor as a daemon. Customize it with the paths to your `uwsgi` binary (e.g. `/usr/local/bin/uwsgi`) and to the configuration file (`~/uwsgi/emperor.ini`). Then move, as superuser, the property list file to its location, load it and start it:

```sh
sudo cp it.unbit.uwsgi.emperor.plist /Library/LaunchDaemons/
sudo launchctl load /Library/LaunchDaemons/it.unbit.uwsgi.emperor.plist
sudo launchctl start /Library/LaunchDaemons/it.unbit.uwsgi.emperor.plist
```

In case you need to stop the daemon:

```sh
sudo launchctl stop /Library/LaunchDaemons/it.unbit.uwsgi.emperor.plist
```

In case you need to unload the daemon:

```sh
sudo launchctl unload /Library/LaunchDaemons/it.unbit.uwsgi.emperor.plist
```