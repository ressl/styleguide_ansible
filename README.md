# Ansible StyleGuide

## TOC
- [Ansible StyleGuide](#ansible-styleguide)
  * [TOC](#toc)
  * [About](#about)
  * [Ansible](#ansible)
    + [Practices](#practices)
      - [Why?](#why-)
    + [Why Doesn't Your Style Follow Theirs?](#why-doesn-t-your-style-follow-theirs-)
  * [Start of Files](#start-of-files)
      - [Why?](#why--1)
    + [End of Files](#end-of-files)
      - [Why?](#why--2)
    + [Quotes](#quotes)
      - [Why?](#why--3)
    + [Environment](#environment)
      - [Why?](#why--4)
    + [Booleans](#booleans)
      - [Why?](#why--5)
    + [Key value pairs](#key-value-pairs)
      - [Why?](#why--6)
    + [Sudo](#sudo)
      - [Why?](#why--7)
    + [Hosts Declaration](#hosts-declaration)
      - [Why?](#why--8)
    + [Task Declaration](#task-declaration)
    + [Why?](#why--9)
  * [Include Declaration](#include-declaration)
      - [Why?](#why--10)
    + [Spacing](#spacing)
      - [Why?](#why--11)
    + [Variable Names](#variable-names)
      - [Why?](#why--12)
    + [bool](#bool)
  * [Lint](#lint)
    + [ansible-lint](#ansible-lint)
    + [yamllint](#yamllint)
    + [Why?](#why--13)
  * [Undo/Rollback Playbook](#undo-rollback-playbook)
    + [Why?](#why--14)
  * [Git Principle](#git-principle)
  * [Git Projects](#git-projects)
  * [README Template](#readme-template)
  * [LICENSE Template](#license-template)
  * [Contributing Template](#contributing-template)
  * [Commit Message Format](#commit-message-format)
  * [Tag Message Format](#tag-message-format)
    + [Rules](#rules)
  * [Notes](#notes)
  * [Git config and GPGsign](#git-config-and-gpgsign)
  * [Tools](#tools)
  * [Contributing](#contributing)
  * [Versioning](#versioning)
  * [Authors](#authors)
  * [License](#license)
  * [Thanks to…](#thanks-to-)

## About

This is an attempt to standardize ansible and the format of commit messages, for the sake of **uniformity** in git log, **best practices** for writing commit messages.

## Ansible

### Practices

You should follow the [Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html) defined by the Ansible documentation when developing playbooks.

#### Why?

The Ansible developers have a good understanding of how the playbooks work and where they look for certain files. Following these practices will avoid a lot of problems.

### Why Doesn't Your Style Follow Theirs?

The script examples are inconsistent in style throughout the Ansible documentation; the purpose of this document is to define a consistent style that can be used throughout Ansible scripts to create robust, readable code.

## Start of Files

You should start your scripts with some comments explaining what the script's purpose does (and an example usage, if necessary), followed by `---` with blank lines around it, then followed by the rest of the script.

```yaml
#bad
- name: 'Change s1m0n3's status'
  service:
    enabled: true
    name: 's1m0ne'
    state: '{{ state }}'
  become: true
  
#good
# Example usage: ansible-playbook -e state=started playbook.yml
# This playbook changes the state of s1m0n3 the robot

---

- name: 'Change s1m0n3's status'
  service:
    enabled: true
    name: 's1m0ne'
    state: '{{ state }}'
  become: true
```

#### Why?

This makes it easier to quickly find out the purpose/usage of a script, either by opening the file or using the `head` command.

### End of Files

You should always end your files with a newline.

#### Why?

This is common Unix best practice, and avoids any prompt misalignment when printing files in a terminal.

### Quotes

**We always quote strings** and prefer single quotes over double quotes. The only time you should use double quotes is when they are nested within single quotes (e.g. Jinja map reference), or when your string requires escaping characters (e.g. using `\n` to represent a newline). If you must write a long string, we use the "folded scalar" style and omit all special quoting. The only things you should avoid quoting are booleans (e.g. true/false), numbers (e.g. 42), and things referencing the local Ansible environemnt (e.g. boolean logic or names of variables we are assigning values to).

```yaml
# bad
- name: start robot named S1m0ne
  service:
    name: s1m0ne
    state: started
    enabled: true
  become: true

# good
- name: 'start robot named S1m0ne'
  service:
    name: 's1m0ne'
    state: 'started'
    enabled: true
  become: true

# double quotes w/ nested single quotes
- name: 'start all robots'
  service:
    name: '{{ item["robot_name"] }}''
    state: 'started'
    enabled: true
  with_items: '{{ robots }}'
  become: true

# double quotes to escape characters
- name 'print some text on two lines'
  debug:
    msg: "This text is on\ntwo lines"

# folded scalar style
- name: 'robot infos'
  debug:
    msg: >
      Robot {{ item['robot_name'] }} is {{ item['status'] }} and in {{ item['az'] }}
      availability zone with a {{ item['curiosity_quotient'] }} curiosity quotient.
  with_items: robots

# folded scalar when the string has nested quotes already
- name: 'print some text'
  debug:
    msg: >
      “I haven’t the slightest idea,” said the Hatter.

# don't quote booleans/numbers
- name: 'download google homepage'
  get_url:
    dest: '/tmp'
    timeout: 60
    url: 'https://google.com'
    validate_certs: true

# variables example 1
- name: 'set a variable'
  set_fact:
    my_var: 'test'

# variables example 2
- name: 'print my_var'
  debug:
    var: my_var
  when: ansible_os_family == 'Darwin'

# variables example 3
- name: 'set another variable'
  set_fact:
    my_second_var: '{{ my_var }}'
```
#### Why?

Even though strings are the default type for YAML, syntax highlighting looks better when explicitly set types. This also helps troubleshoot malformed strings when they should be properly escaped to have the desired effect.

### Environment 

When provisioning a server with environment variables add the environment variables to `/etc/environment` with lineinfile. Do this from the ansible role that is associated with the service or application that is being installed. For example, for Tomcat installation the `CATALINA_HOME` environment variable is often used to reference the folder that contains Tomcat and its associated webapps. 

```yaml
- name: 'add line CATALINA_HOME to /etc/environment'
  lineinfile:
    dest: '/etc/environment'
    line: 'CATALINA_HOME={{ tomcat_home }}'
    state: 'present'
  sudo: true
```

#### Why?
Environment definition files are typically shared so blowing them away by templating them can cause problems. Having the specific environment variable included by `lineinfile` makes it easier to track which applications are dependent upon the environment variable.

### Booleans

```yaml
# bad
- name: 'start sensu-client'
  service:
    name: 'sensu-client'
    state: 'restarted'
    enabled: 1
  become: 'yes'
 
# good
- name: 'start sensu-client'
  service:
    name: 'sensu-client'
    state: 'restarted'
    enabled: true
  become: true
```

#### Why?
There are many different ways to specify a boolean value in ansible, `True/False`, `true/false`, `yes/no`, `1/0`. While it is cute to see all those options we prefer to stick to one : `true/false`. The main reasoning behind this is that Java and JavaScript have similar designations for boolean values. 

### Key value pairs

Use only one space after the colon when designating a key value pair

```yaml
# bad
- name : 'start sensu-client'
  service:
    name    : 'sensu-client'
    state   : 'restarted'
    enabled : true
  become : true


# good
- name: 'start sensu-client'
  service:
    name: 'sensu-client'
    state: 'restarted'
    enabled: true
  become: true
```

**Always use the map syntax,** regardless of how many pairs exist in the map.

```yaml
# bad
- name: 'create checks directory to make it easier to look at checks vs handlers'
  file: 'path=/etc/sensu/conf.d/checks state=directory mode=0755 owner=sensu group=sensu'
  become: true
  
- name: 'copy check-memory.json to /etc/sensu/conf.d'
  copy: 'dest=/etc/sensu/conf.d/checks/ src=checks/check-memory.json'
  become: true
  
# good
- name: 'create checks directory to make it easier to look at checks vs handlers'
  file:
    group: 'sensu'
    mode: '0755'
    owner: 'sensu'
    path: '/etc/sensu/conf.d/checks'
    state: 'directory'
  become: true
  
- name: 'copy check-memory.json to /etc/sensu/conf.d'
  copy:
    dest: '/etc/sensu/conf.d/checks/'
    src: 'checks/check-memory.json'
  become: true
```

#### Why?

It's easier to read and it's not hard to do. It reduces changeset collisions for version control.

### Sudo
Use the new `become` syntax when designating that a task needs to be run with `sudo` privileges

```yaml
# bad
- name: 'template client.json to /etc/sensu/conf.d/'
  template:
    dest: '/etc/sensu/conf.d/client.json'
    src: 'client.json.j2'
  sudo: true
 
# good
- name: 'template client.json to /etc/sensu/conf.d/'
  template:
    dest: '/etc/sensu/conf.d/client.json'
    src: 'client.json.j2'
  become: true
```
#### Why?
Using `sudo` was deprecated at [Ansible version 1.9.1](https://docs.ansible.com/ansible/latest/user_guide/become.html)

### Hosts Declaration

`host` sections should follow this general order:

```yaml
# host declaration
# host options in alphabetical order
# pre_tasks
# roles
# tasks

# example
- hosts: 'webservers'
  remote_user: 'centos'
  vars:
    tomcat_state: 'started'
  pre_tasks:
    - name: 'set the timezone to America/Boise'
      lineinfile:
        dest: '/etc/environment'
        line: 'TZ=America/Boise'
        state: 'present'
      become: true
  roles:
    - { role: 'tomcat', tags: 'tomcat' }
  tasks:
    - name: 'start the tomcat service'
      service:
        name: 'tomcat'
        state: '{{ tomcat_state }}'
```

#### Why?

A proper definition for how to order these maps produces consistent and easily readable code.

### Task Declaration

A task should be defined in such a way that it follows this general order:

```yaml
# task name
# tags
# task map declaration (e.g. service:)
# task parameters in alphabetical order (remember to always use multi-line map syntax)
# loop operators (e.g. with_items)
# task options in alphabetical order (e.g. become, ignore_errors, register)

# example
- name: 'create some ec2 instances'
  tags: 'ec2'
  ec2:
    assign_public_ip: true
    image: 'ami-c7d092f7'
    instance_tags:
      Name: '{{ item }}'
    key_name: 'my_key'
  with_items: '{{ instance_names }}'
  ignore_errors: true
  register: ec2_output
  when: ansible_os_family == 'Darwin'
```

### Why?

Similar to the hosts definition, having a well-defined style here helps us create consistent code.

## Include Declaration

For `include` statements, make sure to quote filenames and only use blank lines between `include` statements if they are multi-line (e.g. they have tags).

```yaml
# bad
- include: other_file.yml

- include: 'second_file.yml'

- include: third_file.yml tags=third

# good

- include: 'other_file.yml'
- include: 'second_file.yml'

- include: 'third_file.yml'
  tags: 'third'
```

#### Why?

This tends to be the most readable way to have `include` statements in your code.

### Spacing

You should have blank lines between two host blocks, between two task blocks, and between host and include blocks. When indenting, you should use 2 spaces to represent sub-maps, and multi-line maps should start with a `-`). For a more in-depth example of how spacing (and other things) should look, consult [style.yml](style.yml).

#### Why?

This produces nice looking code that is easy to read.

### Variable Names

Use `snake_case` for variable names in your scripts.

```yaml
# bad
- name: 'set some facts'
  set_fact:
    myBoolean: true
    myint: 20
    MY_STRING: 'test'

# good
- name: 'set some facts'
  set_fact:
    my_boolean: true
    my_int: 20
    my_string: 'test'
```

#### Why?

Ansible uses `snake_case` for module names so it makes sense to extend this convention to variable names.

### bool

For all bool operators **true** or **false** must be used (Lower case). It is not allowed to use **yes** or **no**.

Example:

```yaml
- name: enable nginx service
  service:
    name: nginx
    state: started
    enabled: true
```

## Lint

Use lint checks on all ansible scripts.

### ansible-lint

`ansible-lint` checks playbooks for practices and behaviour that could potentially be improved.

### yamllint

A linter for YAML files. `yamllint` does not only check for syntax validity, but for weirdnesses like key repetition and cosmetic problems such as lines length, trailing spaces, indentation, etc.

### Why?

So that errors are found and a certain standard is maintained.

## Undo/Rollback Playbook

Create an `Undo/Rollback` function for all playbooks/roles.

### Why?

So that all changes can be undone again.

## Git Principle
Please make as many commits as possible. The principle here is small commits. A version tag must then be set for each related change.

* Every change **must** commit.
* All related changes **must** be combined in a new version.
* Every new version **must** be tagged.

## Git Projects

+ Every project name **must** must be lower case.
+ Every project **must** have an easily recognizable name. The blanks in the project name are filled in with the character **_** (snake_case).
+ Every project **must** have a description.
+ Projects that belong together must be marked with a project description. For example: **ansible_example_role/playbook/module**

## README Template

Please use the [README_ansible_template](README_ansible_template.md) for **ansible** projects.

## LICENSE Template

Please use the [GNU Affero General Public License v3.0](LICENSE) for normal projects.

## Contributing Template

Please use the [CONTRIBUTING](CONTRIBUTING.md) for for projects.

## Commit Message Format

All Git Commit Messages **MUST** be signed!

All Git Commit Messages **MUST** meet with this Text Format:
```
Subject
(Only One NewLine)
Message Body
(Only One NewLine)
Ref <###>
```

## Tag Message Format

All Git Tag Messages **MUST** be signed!

All Git Tag Messages **MUST** meet with this Text Format:
```
Release vSemVer
```

### Rules

1. Capitalize the _Subject_.
2. Do not end the _Subject_ line with a period.
3. Message Body **SHOULD** End with _at-least_ One Issue Tracking ID Reference([GitHub Issues](https://github.com/features#issues)/[GitLab Issues](https://docs.gitlab.com/ee/user/project/issues/)/[Phabricator Tasks](http://phacility.com/phabricator/maniphest/)), Ex. `Issue #27`, `Ref T27` or `Ref T27, T56` or `Fixes T8`.
It's also [recommanded](/../../issues/19) to use _Full URL to Issues_, instead of just Issue ID Number; Doing so will ease browsing issues from terminal.
4. Total Characters of the _Subject Line_ **MUST** be _Less_ than or _Equal_ to **50** Chars Long.
5. Use Valid [MarkDown](https://daringfireball.net/projects/markdown/basics) format in _Message Body_.
6. Use the **Present Tense** ("Add feature" not "Added feature").
7. Use the **Imperative Mood** ("Move cursor to..." not "Moves cursor to...").
8. Use the _Message body_ to explain **what** and **why** vs. how.

## Notes

+ All **WIP** Commits **Should** be Avoided!.
+ Refrencing Issues by using special keywords like `Fixes` or `Resolves` will mark them as closed automatically! For more  information about automatic issue closing using ketwords see: [GitHub](https://help.github.com/articles/closing-issues-via-commit-messages/)/[GitLab](https://docs.gitlab.com/ee/user/project/issues/automatic_issue_closing.html)/[Phabricator](https://secure.phabricator.com/book/phabricator/article/diffusion_autoclose/).
+ There is **NO** New-Line After the _Task ID Reference_ Line.
+ See [ToDo Grammar StyleGuide](https://github.com/slashsBin/styleguide-todo-grammar) for more Information on `@XXX` Comment Tags.

## Git config and GPGsign

All commit messages and tags **must** be signed!

`gpg --list-secret-keys --keyid-format LONG`

```
sec   ed25519 2018-02-26 [SC] [expires: 2022-02-25]
      101D2615211421D8D22218DFD68251B5B79A051A
uid           [ultimate] Robert Ressl <robert.ressl@ibm.com>
ssb   cv25519 2018-02-26 [E] [expires: 2022-02-25]
```

`git config --global user.signingkey 101D2615211421D8D22218DFD68251B5B79A051A`

`git config --global user.name "Ressl Robert"`

`git config --global user.email "robert.ressl@ibm.com"`

`git config --global commit.gpgsign true`

`git config --global tag.gpgsign true`


## Tools

- [ansible](https://www.ansible.com/): Ansible is the simplest way to automate apps and IT infrastructure.
- [yamllint](https://yamllint.readthedocs.io/en/stable/): A linter for YAML files.
- [ansible-lint](https://ansible-lint.readthedocs.io/en/latest/index.html): A linter for ansible files.
- [git](https://git-scm.com/): Git is a free and open source distributed version control system.

## Contributing

Please read [CONTRIBUTING](CONTRIBUTING.md) for details on our code of conduct, and the process for submitting pull requests to us.

## Versioning

* **v1.0.0** - Robert Ressl - Initial work

## Authors

* **Robert Ressl** - *Initial work & improvement* - [Robert Ressl](https://github.com/ressl)

## License

The Code is licensed under the [GNU Affero General Public License v3.0](LICENSE)

## Thanks to…

* [Git StyleGuide](https://github.com/ressl/styleguide_git)
* [Ansible Project](https://docs.ansible.com/) and the [Tips and tricks](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)
* [slashsBin](https://github.com/slashsBin) and his [styleguide-git-commit-message](https://github.com/slashsBin/styleguide-git-commit-message)
* [Coraline Ada Ehmke](https://www.contributor-covenant.org) and his [Contributor Covenant Code of Conduct](http://contributor-covenant.org/version/1/4/)
* [WhiteCloud Analytics](https://github.com/whitecloud) and his [Ansible Styleguide](https://github.com/whitecloud/ansible-styleguide)
