# uWSGI Vassal

Creates an app user, applications directory, virtualenv for an application meant to be run as
a vassal to a uWSGI process in emperor mode.

This role only works with Ubuntu, and it has only been tested on 14.04 and 16.04.

## Usage

### Variables

#### Required

There is one required variable that needs to be defined in your playbook: `app_name`.

#### Configurable

| name              | default                | description                                          |
| ----------------- | ---------------------- | ---------------------------------------------------- |
| python_version    | 3.5                    | python version                                       |
| system_packages   | []                     | a list of apt installable package names              |
| app_user          | www-data               | the name of the user to create                       |
| app_group         | www-data               | the user's group                                     |
| apps_dir          | /www                   | the base path for all the vassal applications        |
| source_dir        | {apps_dir}/{app_name}  | where the application code will live                 |
| venvs_dir         | /var/venvs             | the base path to application virtualenvs             |
| virtualenv_dir    | {venvs_dir}/{app_name} | where the application's virtualenv will libe         |
| python_packages   | []                     | a list of pip installable package names              |
| requirements_file | null                   | the path to the application requirements.txt file    |
| app_enabled       | no                     | should a vassal ini file be added to the emperor     |
| env_vars          | []                     | environment vars to be written to the vassal ini     |
| uwsgi             | {}                     | additional key/value pairs to be added to vassal ini |

### Playbook Examples

#### Minimal

```yaml
- hosts: all
  become: yes
  become_user: root
  roles:
    - role: uwsgi-vassal
      app_name: metalsnake
      app_enabled: yes
```

This will create a virtualenv at `/var/venvs/metalsnake` and an application dir at `/www/metalsnake`
both owned by `www-data:www-data`.

It will also create `/etc/uwsgi-emperor/vassals/metalsnake.ini` vassal config file:

```ini
[uwsgi]
home = /var/venvs/metalsnake
pythonpath = /www/metalsnake
chdir = /www/metalsnake
```

#### Configured - Flask App

```yaml
- hosts: all
  become: yes
  become_user: root
  roles:
    - role: uwsgi-vassal
      app_name: metalsnake
      app_enabled: yes
      requirements_file: /www/metalsnake/requirements.txt
      system_packages:
        - libpq-dev
        - libssl-dev
      uwsgi:
        wsgi-file: app.py
        callable: app
        socket: 127.0.0.1:5000
        workers: 1
        buffer-size: 16384
```

This will create a virtualenv at `/var/venvs/metalsnake` and an application dir at `/www/metalsnake`
both owned by `www-data:www-data`.

It will also create `/etc/uwsgi-emperor/vassals/metalsnake.ini` vassal config file:

```ini
[uwsgi]
wsgi-file = app.py
callable = app
socket: 127.0.0.1:5000
buffer-size = 16384
home = /var/venvs/metalsnake
pythonpath = /www/metalsnake
chdir = /www/metalsnake
```

#### Configured - Django App

```yaml
- hosts: all
  become: yes
  become_user: root
  roles:
    - role: uwsgi-vassal
      app_name: metalsnake
      app_enabled: yes
      requirements_file: /www/metalsnake/requirements.txt
      system_packages:
        - libpq-dev
        - libssl-dev
      uwsgi:
        module: metalsnake.wsgi:application
        socket: 127.0.0.1:8000
        workers: 1
        buffer-size: 16384
        pythonpath: /www/metalsnake/metalsnake
        chdir: /www/metalsnake/metalsnake
      env_vars:
        - "DJANGO_SETTINGS_MODULE=metalsnake.settings.vagrant"
        - "PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games"
```

This will create a virtualenv at `/var/venvs/metalsnake` and an application dir at `/www/metalsnake`
both owned by `www-data:www-data`.

It will also create `/etc/uwsgi-emperor/vassals/metalsnake.ini` vassal config file:

```ini
[uwsgi]
module = metalsnake.wsgi:application
socket = 127.0.0.1:8000
buffer-size = 16384
home = /var/venvs/metalsnake
pythonpath = /www/metalsnake/metalsnake
chdir = /www/metalsnake/metalsnake
env = DJANGO_SETTINGS_MODULE=metalsnake.settings.vagrant
env = PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
```




















