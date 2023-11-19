# Project Playbook/Role/Module

One Paragraph of project description goes here

## Requirements

### Role/Modules

The following roles are direct dependencies because they're used for common "default" functionality.

- `example` description

### Platforms

The following platforms are supported and tested with Test Kitchen:

- RHEL 8
- Ubuntu 16.04+
- CentOS 7+
- Debian 10.7+
- openSUSE
- FreeBSD

Other Debian and RHEL family distributions are assumed to work.

### Ansible

- Ansible 2.9+

## Variables

Variables for this playbook/role are logically separated into different files. Some variables are set only via a specific file.

### project::option

These variables are used in the `project::option`.

```yml
project:
  option1: one
  option2: two
```

## Resources

### resource_name

Explain what these tests test and why

### Actions

- `enable` - Enable ...
- `disable` - Disable ...

### Properties:

- `example` - description

### Package installation

Explain what these tests test and why

#### Specifying Modules to compile

Explain what these tests test and why

## Authors

* **Robert Ressl** - *Initial work* - [Robert Ressl](https://github.com/ressl)

See also the list of [contributors](https://bitbucketenterprise.aws.novartis.net/your/project/contributors) who participated in this project.

## License

This project is licensed under the GNU Affero General Public License v3.0 License - see the [LICENSE](LICENSE) file for details

## Acknowledgments

* Hat tip to anyone who's code was used
* Inspiration
* etc

## Thanks to…

* People who helped you.
* Persons you have taken code from
