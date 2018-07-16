# Definition

This is a step-by-step guide to install and configure the uWSGI server in _emperor_ mode and with multiple language support. It is mainly intended for macOS and Ubuntu environments but it can be adapted to other platforms.

# Configuration

Please make sure to read each of the following sections, and complete them in the specified order.

### uWSGI

uWSGI will be installed from its sources with the support for multiple languages, then a plugin will be built for each language to be supported:
- get uwsgi latest sources from [here](http://uwsgi-docs.readthedocs.io/en/latest/Download.html);
- extract the files in a local directory;
- open a terminal and `cd` into the uwsgi sources directory;
- build the uwsgi binary:
```sh
make nolang
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
- compile the bonjour uWSGI plugin:

```sh
uwsgi --build-plugin https://github.com/unbit/uwsgi-bonjour
```
- copy the plugin to the uWSGI plugins directory:

```sh
cp bonjour_plugin.so ~/uwsgi/plugins/
```
Bonjour works thanks to Unbit uwsgi-bonjour, for further information check [here](https://github.com/unbit/uwsgi-bonjour).

### Avahi

This part is only meant for GNU/Linux and makes it possible to assign a custom domain to each project (e.g. `my_new_project.local/`):
- install the libavahi-client development headers (e.g. with apt):

```sh
sudo apt install libavahi-client-dev
```
- compile the avahi uWSGI plugin:

```sh
uwsgi --build-plugin https://github.com/20tab/uwsgi-avahi
```
- copy the plugin to the uWSGI plugins directory:

```sh
cp avahi_plugin.so ~/uwsgi/plugins/
```
Avahi works thanks to 20tab uwsgi-avahi, for further information check [here](https://github.com/20tab/uwsgi-avahi).

### Emperor

For more information on uWSGI _emperor_ mode refer to the [official documentation](http://uwsgi-docs.readthedocs.io/en/latest/Emperor.html). Before configuring uWSGI:
- create a directory where to collect all the projects .ini configuration files (e.g. `~/uwsgi/vassals/`);

Provided `emperor.ini` contains a sample configuration to run the uWSGI server in _emperor_ mode. Customize it with the paths to your vassals and plugins directories and to the emperor log file, plus any other setting/option you wish to run the emperor with. Then:
- move the customised emperor configuration file to a convenient location:

```sh
cp emperor.ini ~/uwsgi/
```
Be careful not to put the emperor configuration file into the vassals configuration files directory. Also note that the provided configuration file sets the emperor to use port 80, which must be available for the server to work.

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

##### Avahi (GNU/Linux)

In order to assign a custom domain to a project (e.g. `my_new_project.local/`), also include the following lines in the corresponding vassal configuration file:

```INI
plugin = avahi
avahi-register = %(project_name).local
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

### Automatic (Ubuntu >= 15.04)

Provided `emperor.uwsgi.service`, customize it with the paths to your `uwsgi` binary (e.g. `/usr/local/bin/uwsgi`) and to the configuration file (`~/uwsgi/emperor.ini`). Then move, as superuser, the service file to its location, load it and start it:

```sh
sudo cp emperor.uwsgi.service /etc/systemd/system/
sudo systemctl start emperor.uwsgi.service
sudo systemctl enable emperor.uwsgi.service
```

In case you need to stop the daemon:

```sh
sudo systemctl stop emperor.uwsgi.service
```

In case you need to disable the daemon:

```sh
sudo systemctl disable emperor.uwsgi.service
```

##### For other distros check [here](https://uwsgi-docs.readthedocs.io/en/latest/Management.html).
