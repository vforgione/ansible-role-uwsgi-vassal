# uWSGI Vassal

Creates an app user, applications directory, virtualenv for an application meant to be run as
a vassal to a uWSGI process in emperor mode.

## Usage

### Variables

There is one required variable that needs to be defined in your playbook: `app_name`.

There are a number of other configurable variables:

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

### Minimum Playbook

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
