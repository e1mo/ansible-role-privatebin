# ansible-role-privatebin

Ansible role to setup the PHP-based [PrivateBin](https://github.com/PrivateBin/PrivateBin)

## Requirements

Those are the minimal requirements for PrivateBin (as of Version 1.3.4):

- PHP version 5.5 or above
- _one_ of the following sources of cryptographically safe randomness is required:
  - PHP 7 or higher
  - [Libsodium](https://doc.libsodium.org/installation/) and it's [PHP extension](https://paragonie.com/book/pecl-libsodium/read/00-intro.md#installing-libsodium)
  - open_basedir access to `/dev/urandom`
  - mcrypt extension
  - com_dotnet extension

  Mcrypt needs to be able to access `/dev/urandom`. This means if `open_basedir` is set, it must include this file.
- GD extension
- some disk space or (optionally) a database supported by [PDO](https://secure.php.net/manual/book.pdo.php)
- ability to create files and folders in the installation directory and the PATH defined in index.php
- A web browser with javascript support

Taken from the [PrivateBin wiki](https://github.com/PrivateBin/PrivateBin/blob/master/INSTALL.md#installation), mostly written by [elrido](https://github.com/elrido).

## Role Variables

| Variable                          	| Description                                                                                                                            	| Default                                                                                                        	|
|-----------------------------------	|----------------------------------------------------------------------------------------------------------------------------------------	|----------------------------------------------------------------------------------------------------------------	|
| `pbin_path`                       	| Location of the PrivateBin source files                                                                                                	| `"/var/www/privatebin"`                                                                                        	|
| `pbin_user`                       	| User to be the owner of the PrivateBin files                                                                                           	| `"{{ ansible_facts['user_id'] }}"` (user executing the tasks on the remote machine)                            	|
| `pbin_group`                      	| Group to be the owner of the PrivateBin files                                                                                          	| `"{{ pbin_use }}"`                                                                                             	|
| `pbin_git_repo`                   	| Git repository to clone                                                                                                                	| `"https://github.com/PrivateBin/PrivateBin.git"`                                                               	|
| `pbin_git_version`                	| Git version (e.g. branch name or tag) to clone                                                                                         	| `"1.3.4"` (latest as of writing this role)                                                                     	|
| `pbin_model_class`                	| `Filesystem` or `Database`, where to store pastes                                                                                      	| `"Filesystem"`                                                                                                 	|
| `pbin_datadir`                    	| Folder to store pastes in, applicable when `Filesystem`                                                                                	| `"data"`                                                                                                       	|
| `pbin_pdo_dsn`                    	| DSN string to use for Database connection (see <https://www.php.net/manual/en/pdo.drivers.php> for reference)                          	| `""` (empty)                                                                                                   	|
| `pbin_pdo_table`                  	| Table prefix in MySQL / PsQL / SQLite3 / ...                                                                                           	| `"privatebin_"`                                                                                                	|
| `pbin_pdo_user`                   	| Username for authenticating against the database                                                                                       	| `""` (empty)                                                                                                   	|
| `pbin_name`                       	| Name of the PrivateBin installation                                                                                                    	| `"PrivateBin"`                                                                                                 	|
| `pbin_discussion_enabled`         	| Allow discussions to be opened                                                                                                         	| `true`                                                                                                         	|
| `pbin_password_enabled`           	| Allow custom passwords to be set                                                                                                       	| `true`                                                                                                         	|
| `pbin_fileupload_enabled`         	| Allow files to be attached to pastes                                                                                                   	| `true`                                                                                                         	|
| `pbin_burn_after_reading_default` 	| Set the checkmark to delete pastes after reading by default                                                                            	| `false`                                                                                                        	|
| `pbin_formatter_default`          	| Default formatter to use (`plaintext`, `markdown` or `syntaxhighlighting`)                                                             	| `"plaintext"`                                                                                                  	|
| `pbin_syntax_theme`               	| Theme to use for syntax highlighting, `false` to apply no custom theme                                                                 	| `false`                                                                                                        	|
| `pbin_template`                   	| Frontend template to use                                                                                                               	| `"bootstrap"`                                                                                                  	|
| `pbin_language_selection`         	| Display the language selection dropdown                                                                                                	| `false`                                                                                                        	|
| `pbin_sizelimit`                  	| Limit for the size of each paste in bytes                                                                                              	| `10485760` (10 Mebibytes)                                                                                      	|
| `pbin_notice`                     	| Add a notice to the privatebin frontend, false to disable                                                                              	| `false`                                                                                                        	|
| `pbin_formatter_options`          	| Set available formatters, their order and their labels                                                                                 	| `[plaintext: "Plain Text", syntaxhighlighting: "Source Code", markdown: "Markdown"]`                           	|
| `pbin_compression`                	| Compression method to use, `zlib` or `none`                                                                                            	| `"zlib"`                                                                                                       	|
| `pbin_expire_default`             	| Default expiry time for pastes, must be present in `pbin_expire_options`                                                               	| `"1week"`                                                                                                      	|
| `pbin_expire_options`             	| Available expiration times in second                                                                                                   	| `[5min: 300, 10min: 600, 1hour: 3600, 1day: 86400, 1week: 604800, 1month: 2592000, 1year: 31536000, never: 0]` 	|
| `pbin_ratelimit`                  	| Seconds in between pates from the same IP                                                                                              	| `10`                                                                                                           	|
| `pbin_forwarded_header`           	| If running behind a reverse proxy, set to the name of the header containing the clients IP such as `X_FORWARDED_FOR`, false to disable 	| `false`                                                                                                        	|
| `pbin_traffic_dir`                	| Directory to store the traffic limits in                                                                                               	| `"{{ pbin_datadir }}"`                                                                                         	|
| `pbin_purge_limit`                	| Minimum time between purging attempts in seconds                                                                                       	| `300`                                                                                                          	|
| `pbin_purge_batchsize`            	| Maximum number of pastes to delete when purging, larger installations may need to increase this value                                  	| `10`                                                                                                           	|
| `pbin_purge_dir`                  	| Directory to store the purge limit in                                                                                                  	| `"{{ pbin_datadir }}"`                                                                                         	|

## Example Playbook

```yaml
- name: Install PrivateBin
  hosts: privatebin
  roles:
    - e1mo.privatebin
  tags:
    - privatebin
  vars:
    pbin_path: "/var/www/bin.e1mo.de"
    pbin_user: "www-data"
    pbin_model_class: "Filesystem"
    pbin_pdo_dsn: "mysql:host=localhost;dbname=privatebin"
    pbin_pdo_user: "privatebin"
    pbin_pdo_pass: "privatebin"
```

## License

BSD-3-Clause

## Related projects

A selection of selected Projects that relate to this role, helped to create or inspired me:

- [PrivateBin](https://privatebin.info/): The awesome PrivateBin project itself.
- [ansible-role-php](https://github.com/geerlingguy/ansible-role-php): A ansible role from [Jeff Geerling aka. geerlingguy](https://github.com/geerlingguy) to install PHP, MySQL and PostreSQL respectively on your host. He also has a entertaining and educative [Youtube channel](https://www.youtube.com/user/geerlingguy)
- [ansible](https://github.com/ansible/ansible): The ansible project itself, without which this role could not exist.

## Author Information

Written by [Moritz 'e1mo' Fromm](https://e1mo.de).

The role is developed on [sourcehut](https://sr.ht) at <https://git.sr.ht/~e1mo/ansible-role-privatebin>. To contribute send your patches to `~e1mo/ansible-role-privatebin [at] lists.sr.ht` using [`git send-email`](https://git-send-email.io/) ([Mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)). The issue-tracker is located at <https://todo.sr.ht/~e1mo/ansible-role-privatebin>, no account needed.
