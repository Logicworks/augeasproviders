---
layout: minimal
title: augeasproviders - Examples
---

# Manifest examples

Examples are given below for each of the providers and custom types in
`augeasproviders`.

* [apache_directive provider](#apachedirective-provider)
* [apache_setenv provider](#apachesetenv-provider)
* [host provider](#host-provider)
* [kernel_parameter provider](#kernelparameter-provider)
* [mailalias provider](#mailalias-provider)
* [mounttab provider](#mounttab-provider)
* [nrpe_command provider](#nrpecommand-provider)
* [pam provider](#pam-provider)
* [pg_hba provider](#pghba-provider)
* [puppet_auth provider](#puppetauth-provider)
* [shellvar provider](#shellvar-provider)
* [ssh_config provider](#sshconfig-provider)
* [sshd_config provider](#sshdconfig-provider)
* [sshd_config_subsystem provider](#sshdconfigsubsystem-provider)
* [sshkey provider](#sshkey-provider)
* [sysctl provider](#sysctl-provider)
* [syslog provider](#syslog-provider)


## apache_directive provider

This is a custom type and provider supplied by `augeasproviders`.

### manage simple entry

    apache_directive { "StartServers":
      args   => 4,
      ensure => present,
    }

### delete entry

    apache_directive { "ServerName":
      args   => "foo.example.com",
      ensure => absent,
    }

### manage entry in another config location

    apache_directive { "SetEnv":
      args        => ["SPECIAL_PATH", "/foo/bin"],
      args_params => 1,
      ensure      => present,
      target      => "/etc/httpd/conf.d/app.conf",
    }

The `SetEnv` directive is not unique per scope: the first arg identifies the entry we want to update, and needs to be taken into account. For this reason, we set `args_params` to `1`.

### set a value in a given context

    apache_directive { 'StartServers for mpm_prefork_module':
      ensure      => present,
      name        => 'StartServers',
      context     => 'IfModule[arg="mpm_prefork_module"]',
      args        => 4,
    }


The directive is nested in the context of the `mpm_prefork_module` module, so we specify this with the `context` parameter.

The value of `StartServers` for the `mpm_prefork_module` module will be set/updated to `4`. Note that the `IfModule` entry will not be created if it is missing.

## apache_setenv provider

This is a custom type and provider supplied by `augeasproviders`.

### manage simple entry

    apache_setenv { "SPECIAL_PATH":
      ensure => present,
      value  => "/foo/bin",
    }

### manage entry with no value

    apache_setenv { "ENABLE_FOO":
      ensure  => present,
    }

### delete entry

    apache_setenv { "SPECIAL_PATH":
      ensure => absent,
    }

### manage entry in another config location

    apache_setenv { "SPECIAL_PATH":
      ensure => present,
      value  => "/foo/bin",
      target => "/etc/httpd/conf.d/app.conf",
    }

## host provider

This is a provider for a type distributed in Puppet core: [host type
reference](http://docs.puppetlabs.com/references/stable/type.html#host).

The provider needs to be explicitly given as `augeas` to use `augeasproviders`.

The `comment` parameter is only supported on Puppet 2.7 and higher.

### manage simple entry

    host { "example":
      ensure   => present,
      ip       => "192.168.1.1",
      provider => augeas,
    }

### manage entry with aliases and comment

    host { "example":
      ensure       => present,
      ip           => "192.168.1.1",
      host_aliases => [ "foo-a", "foo-b" ],
      comment      => "test",
      provider     => augeas,
    }

### manage entry in another location

    host { "example":
      ensure   => present,
      ip       => "192.168.1.1",
      target   => "/etc/anotherhosts",
      provider => augeas,
    }

### delete entry

    host { "iridium":
      ensure   => absent,
      provider => augeas,
    }

### remove aliases

    host { "iridium":
      ensure       => present,
      host_aliases => [],
      provider     => augeas,
    }

### remove comment

    host { "argon":
      ensure   => present,
      comment  => "",
      provider => augeas,
    }

## kernel_parameter provider

This is a custom type and provider supplied by `augeasproviders`.  It supports
both GRUB Legacy (0.9x) and GRUB 2 configurations.

### manage parameter without value

    kernel_parameter { "quiet":
      ensure => present,
    }

### manage parameter with value

    kernel_parameter { "elevator":
      ensure  => present,
      value   => "deadline",
    }

### manage parameter with multiple values

    kernel_parameter { "rd_LVM_LV":
      ensure  => present,
      value   => ["vg/lvroot", "vg/lvvar"],
    }

### manage parameter on certain boot types

Bootmode defaults to "all", so settings are applied for all boot types usually.

Apply only to normal boots:

    kernel_parameter { "quiet":
      ensure   => present,
      bootmode => "normal",
    }

Only recovery mode boots (unsupported with GRUB 2):

    kernel_parameter { "quiet":
      ensure   => present,
      bootmode => "recovery",
    }

### delete entry

    kernel_parameter { "rhgb":
      ensure => absent,
    }

### manage parameter in another config location

    kernel_parameter { "elevator":
      ensure => present,
      value  => "deadline",
      target => "/mnt/boot/grub/menu.lst",
    }

## mailalias provider

This is a provider for a type distributed in Puppet core: [mailalias type
reference](http://docs.puppetlabs.com/references/stable/type.html#mailalias).

The provider needs to be explicitly given as `augeas` to use `augeasproviders`.

### manage simple entry

    mailalias { "example":
      ensure    => present,
      recipient => "bar",
      provider  => augeas,
    }

### manage entry with multiple recipients

    mailalias { "example":
      ensure    => present,
      recipient => [ "fred", "bob" ],
      provider  => augeas,
    }

### manage entry in another location

    mailalias { "example":
      ensure    => present,
      recipient => "bar",
      target    => "/etc/anotheraliases",
      provider  => augeas,
    }

### delete entry

    mailalias { "mailer-daemon":
      ensure   => absent,
      provider => augeas,
    }

## mounttab provider

This is a provider for a type distributed in the [puppetlabs-mount_providers
module](http://forge.puppetlabs.com/puppetlabs/mount_providers).

The provider needs to be explicitly given as `augeas` to use `augeasproviders`.

If editing a vfstab entry, slightly different options need to be passed
compared to a fstab entry.

### manage simple fstab entry

    mounttab { "/mnt":
      ensure   => present,
      device   => "/dev/myvg/mytest",
      fstype   => "ext4",
      options  => "defaults",
      provider => augeas,
    }

### manage full fstab entry

    mounttab { "/mnt":
      ensure   => present,
      device   => "/dev/myvg/mytest",
      fstype   => "ext4",
      options  => ["nosuid", "uid=12345"],
      dump     => "1",
      pass     => "2",
      provider => augeas,
    }

### manage fstab entry with default options

    mounttab { "/mnt":
      ensure   => present,
      device   => "/dev/myvg/mytest",
      fstype   => "ext4",
      provider => augeas,
    }

### delete fstab entry

    mounttab { "/":
      ensure   => absent,
      provider => augeas,
    }

### manage entry in another fstab location

    mounttab { "/home":
      ensure   => present,
      device   => "/dev/myvg/mytest",
      target   => "/etc/anotherfstab",
      provider => augeas
    }

### manage device in fstab entry only

    mounttab { "/home":
      ensure   => present,
      device   => "/dev/myvg/mytest",
      provider => augeas
    }

Note: dump and pass are both changing unless explicitly specified, see issue
[#16122](http://projects.puppetlabs.com/issues/16122).

### manage fstype in fstab entry only

    mounttab { "/home":
      ensure   => present,
      fstype   => "btrfs",
      provider => augeas,
    }

### manage options in fstab entry only

    mounttab { "/home":
      ensure   => present,
      options  => "nosuid",
      provider => augeas,
    }

### manage complex options in fstab entry only

    mounttab { "/home":
      ensure   => present,
      options  => [
        "nosuid",
        "uid=12345",
        'rootcontext="system_u:object_r:tmpfs_t:s0"',
      ],
      provider => augeas,
    }

### remove options from fstab entry

    mounttab { "/home":
      ensure   => present,
      options  => [],
      provider => augeas,
    }

### manage simple vfstab entry

    mounttab { "/mnt":
      ensure   => present,
      device   => "/dev/dsk/c1t1d1s1",
      fstype   => "ufs",
      atboot   => "yes",
      provider => augeas,
    }

### manage full vfstab entry

    mounttab { "/mnt":
      ensure      => present,
      device      => "/dev/dsk/c1t1d1s1",
      blockdevice => "/dev/foo/c1t1d1s1",
      fstype      => "ufs",
      pass        => "2",
      atboot      => "yes",
      options     => [ "nosuid", "nodev" ],
      provider    => augeas,
    }

### manage vfstab entry with default options

    mounttab { "/mnt":
      ensure   => present,
      device   => "/dev/myvg/mytest",
      fstype   => "ext4",
      provider => augeas,
    }

### delete vfstab entry

    mounttab { "/":
      ensure   => absent,
      provider => augeas,
    }

### remove options from vfstab entry

    mounttab { "/home":
      ensure   => present,
      options  => [],
      provider => augeas,
    }

## nrpe_command provider

This is a custom type and provider supplied by `augeasproviders`.

### manage entry

    nrpe_command { "check_spec_test":
      ensure  => present,
      command => "/usr/bin/check_my_thing -p 'some command with \"multiple [types]\" of quotes' -x and-stuff",
    }

### delete entry

    nrpe_command { "check_test":
      ensure => absent,
    }

## pam provider

This is a custom type and provider supplied by `augeasproviders`.

### manage simple entry

    pam { "Set sss entry to system-auth auth":
      ensure    => present,
      service   => 'system-auth',
      type      => 'auth',
      control   => 'sufficient',
      module    => 'pam_sss.so',
      arguments => 'use_first_pass',
      position  => 'before module pam_deny.so',
    }

### manage same entry but with Augeas xpath

    pam { "Set sss entry to system-auth auth":
      ensure    => present,
      service   => 'system-auth',
      type      => 'auth',
      control   => 'sufficient',
      module    => 'pam_sss.so',
      arguments => 'use_first_pass',
      position  => 'before *[type="auth" and module="pam_deny.so"]',
    }

### delete entry

    pam { "Remove sss auth entry from system-auth":
      ensure  => absent,
      service => 'system-auth',
      type    => 'auth',
      module  => 'pam_sss.so',
    }

### delete all references to module in file

    pam { "Remove all pam_sss.so from system-auth":
      ensure  => absent,
      service => 'system-auth',
      module  => 'pam_sss.so',
    }

### manage entry in another pam service

    pam { "Set cracklib limits in password-auth":
      ensure    => present,
      service   => 'password-auth',
      type      => 'password',
      module    => 'pam_cracklib.so',
      arguments => ['try_first_pass','retry=3', 'minlen=10'],
    }

### manage entry like previous but in classic pam.conf

    pam { "Set cracklib limits in password-auth":
      ensure    => present,
      service   => 'password-auth',
      type      => 'password',
      module    => 'pam_cracklib.so',
      arguments => ['try_first_pass','retry=3', 'minlen=10'],
      target    => '/etc/pam.conf',
    }

### allow multiple entries with same control value

    pam { "Set invalid login 3 times deny in password-auth -fail":
      ensure           => present,
      service          => 'password-auth',
      type             => 'auth',
      control          => '[default=die]',
      control_is_param => true,
      module           => 'pam_faillock.so',
      arguments        => ['authfail','deny=3','unlock_time=604800','fail_interval=900'],
    }

## pg_hba provider

This is a custom type and provider supplied by `augeasproviders`.

### Composite namevars

This type supports composite namevars in order to easily specify the entry you want to manage. The format for composite namevars is:

    local to <user> on <database> [in <target>]

if defining a local (socket) rule, or:

    <type> to <user> on <database> from <address> [in <target>]

otherwise.

In each form, `in <target>` is optional. You can also use a personalized namevar and specify all parameters manually.


### manage simple local entry

    pg_hba { 'local to all on all':
      ensure => present,
      method => 'md5',
      target => '/etc/postgresql/9.1/main/pg_hba.conf',
    }

### manage simple host entry

    pg_hba { 'host to all on all from 192.168.0.1':
      ensure => present,
      method => 'md5',
      target => '/etc/postgresql/9.1/main/pg_hba.conf',
    }

### multiple users and databases

    pg_hba { 'host to user1,user2 on db1,db2 from 192.168.0.1':
      ensure => present,
      method => 'md5',
      target => '/etc/postgresql/9.1/main/pg_hba.conf',
    }

    pg_hba { 'Allow +foo and @bar to mydb and yourdb':
      ensure   => present,
      user     => ['+foo', '@bar'],
      database => ['mydb', 'yourdb'],
      method   => 'md5',
      target   => '/etc/postgresql/9.1/main/pg_hba.conf',
    }

### using a personalized namevar

    pg_hba { 'Default entry':
      type     => 'local',
      user     => 'all',
      database => 'all',
      method   => 'md5',
      target   => '/etc/postgresql/9.1/main/pg_hba.conf',
    }

### pass options for the method

    pg_hba { 'Default entry with option':
      method  => 'ident',
      options => { 'sameuser' => undef },
      target  => '/etc/postgresql/9.1/main/pg_hba.conf',
    }

    pg_hba { 'host to all on all from .dev.example.com in /etc/postgresql/9.1/main/pg_hba.conf':
      method  => 'ldap',
      options => {
        'ldapserver' => 'auth.example.com',
        'ldaptls'    => '1',
        'ldapprefix' => 'uid=',
        'ldapsuffix' => ',ou=people,dc=example,dc=com',
      },
    }

### insert entry in specific position

    pg_hba { 'local to all on all':
      ensure   => present,
      method   => 'md5',
      position => 'before first entry',
      target   => '/etc/postgresql/9.1/main/pg_hba.conf',
    }

    pg_hba { 'local to all on all':
      ensure   => present,
      method   => 'md5',
      position => 'after last entry',
      target   => '/etc/postgresql/9.1/main/pg_hba.conf',
    }

    pg_hba { 'local to all on all':
      ensure   => present,
      method   => 'md5',
      position => 'before last local',
      target   => '/etc/postgresql/9.1/main/pg_hba.conf',
    }

    pg_hba { 'local to all on all':
      ensure   => present,
      method   => 'md5',
      position => 'after first hostssl',
      target   => '/etc/postgresql/9.1/main/pg_hba.conf',
    }

    pg_hba { 'local to all on all':
      ensure   => present,
      method   => 'md5',
      position => 'after first anyhost', # any type matching host.*
      target   => '/etc/postgresql/9.1/main/pg_hba.conf',
    }

    pg_hba { 'local to all on all':
      ensure   => present,
      method   => 'md5',
      position => 'before 5', # Before the fifth entry
      target   => '/etc/postgresql/9.1/main/pg_hba.conf',
    }

    pg_hba { 'local to all on all':
      ensure   => present,
      method   => 'md5',
      position => '*[database="all" and user="admin"][1]', # First entry for database 'all' and user 'admin'
      target   => '/etc/postgresql/9.1/main/pg_hba.conf',
    }

### ensure position is correct

    pg_hba { 'local to all on all':
      ensure   => positioned,
      method   => 'md5',
      position => 'before first entry',
      target   => '/etc/postgresql/9.1/main/pg_hba.conf',
    }

### delete entry

    pg_hba { 'local to all on all':
      ensure => absent,
      target => '/etc/postgresql/9.1/main/pg_hba.conf',
    }

    pg_hba { 'host to all on all from 192.168.0.1':
      ensure    => absent,
      target => '/etc/postgresql/9.1/main/pg_hba.conf',
    }

## puppet_auth provider

This is a custom type and provider supplied by `augeasproviders`.

It requires the `Puppet_Auth.lns` lens, which is provided with versions of Augeas strictly greater than 0.10.0.

### manage simple entry

    puppet_auth { 'Deny /facts':
      ensure        => present,
      path          => '/facts',
      authenticated => 'any',
    }

### manage regex entry

    puppet_auth { 'Deny ~ ^/facts/([^/]+)$':
      ensure        => present,
      path          => '^/facts/([^/]+)$',
      path_regex    => true,
      authenticated => 'any',
    }

### add multiple environments

    puppet_auth { 'Allow /facts for prod and dev environments from same client':
      ensure        => present,
      path          => '/facts',
      authenticated => 'any',
      allow         => '$1',
      environments  => ['prod', 'dev'],
    }

### ensure an entry is before a given path

`ins_after` provides the opposite functionality, so an entry is created after a
given path.

    puppet_auth { 'Allow /facts before first denied rule':
      ensure        => present,
      path          => '/facts',
      authenticated => 'any',
      allow         => '*',
      ins_before    => 'first deny',
    }

### delete entry

    puppet_auth { 'Remove /facts':
      ensure => absent,
      path   => '/facts',
    }

## shellvar provider

This is a custom type and provider supplied by `augeasproviders`.

### manage simple entry

    shellvar { "HOSTNAME":
      ensure => present,
      target => "/etc/sysconfig/network",
      value  => "host.example.com",
    }

    shellvar { "disable rsyncd":
      ensure   => present,
      target   => "/etc/default/rsync",
      variable => "RSYNC_ENABLE",
      value    => "false",
    }

    shellvar { "ntpd options":
      ensure   => present,
      target   => "/etc/sysconfig/ntpd",
      variable => "OPTIONS",
      value    => "-g -x -c /etc/myntp.conf",
    }

### manage entry with comment

    shellvar { "HOSTNAME":
      ensure  => present,
      target  => "/etc/sysconfig/network",
      comment => "My server's hostname",
      value   => "host.example.com",
    }

### export values

    shellvar { "HOSTNAME":
      ensure  => exported,
      target  => "/etc/sysconfig/network",
      value   => "host.example.com",
    }

### unset values

    shellvar { "HOSTNAME":
      ensure  => unset,
      target  => "/etc/sysconfig/network",
    }

### force quoting style

Values needing quotes will automatically get them, but they can also be
explicitly enabled.  Unfortunately the provider doesn't help with quoting the
values themselves.

    shellvar { "RSYNC_IONICE":
      ensure   => present,
      target   => "/etc/default/rsync",
      value    => "-c3",
      quoted   => "single",
    }

### delete entry

    shellvar { "RSYNC_IONICE":
      ensure => absent,
      target => "/etc/default/rsync",
    }

### remove comment from entry

    shellvar { "HOSTNAME":
      ensure  => present,
      target  => "/etc/sysconfig/network",
      comment => "",
    }

### replace commented value with entry

    shellvar { "HOSTNAME":
      ensure    => present,
      target    => "/etc/sysconfig/network",
      comment   => "",
      uncomment => true,
    }

### array values

You can pass array values to the type.

There are two ways of rendering array values, and the behavior is set using
the `array_type` parameter. `array_type` takes three possible values:

* `auto` (default): detects the type of the existing variable, defaults to `string`;
* `string`: renders the array as a string, with a space as element separator;
* `array`: renders the array as a shell array.

For example:

    shellvar { "PORTS":
      ensure     => present,
      target     => "/etc/default/puppetmaster",
      value      => ["18140", "18141", "18142"],
      array_type => "auto",
    }

will create `PORTS="18140 18141 18142"` by default, and will change `PORTS=(123)` to `PORTS=("18140" "18141" "18142")`.

    shellvar { "PORTS":
      ensure     => present,
      target     => "/etc/default/puppetmaster",
      value      => ["18140", "18141", "18142"],
      array_type => "string",
    }

will create `PORTS="18140 18141 18142"` by default, and will change `PORTS=(123)` to `PORTS="18140 18141 18142"`.

    shellvar { "PORTS":
      ensure     => present,
      target     => "/etc/default/puppetmaster",
      value      => ["18140", "18141", "18142"],
      array_type => "array",
    }

will create `PORTS=("18140" "18141" "18142")` by default, and will change `PORTS=123` to `PORTS=(18140 18141 18142)`.

Quoting is honored for arrays:

* When using the string behavior, quoting is global to the string;
* When using the array behavior, each value in the array is quoted as requested.

### appending to arrays

    shellvar { "GRUB_CMDLINE_LINUX":
      ensure       => present,
      target       => "/etc/default/grub",
      value        => "cgroup_enable=memory",
      array_append => true,
    }

will change `GRUB_CMDLINE_LINUX="quiet splash"` to `GRUB_CMDLINE_LINUX="quiet splash cgroup_enable=memory"`.

    shellvar { "GRUB_CMDLINE_LINUX":
      ensure       => present,
      target       => "/etc/default/grub",
      value        => ["quiet", "cgroup_enable=memory"],
      array_append => true,
    }

will also change `GRUB_CMDLINE_LINUX="quiet splash"` to `GRUB_CMDLINE_LINUX="quiet splash cgroup_enable=memory"`.

## ssh_config provider

This is a custom type and provider supplied by `augeasproviders`.

### manage simple entry

    ssh_config { "ForwardAgent":
      ensure => present,
      value  => "yes",
    }

### manage array entry

    ssh_config { "SendEnv":
      ensure => present,
      value  => ["LC_*", "LANG"],
    }

### manage entry for a specific host

    ssh_config { "X11Forwarding":
      ensure    => present,
      host      => "example.net",
      value     => "yes",
    }

### manage entries with same name for different hosts

    ssh_config { "ForwardAgent global":
      ensure => present,
      key    => "ForwardAgent",
      value  => "no",
    }

    ssh_config { "ForwardAgent on example.net":
      ensure    => present,
      key       => "ForwardAgent",
      host      => "example.net",
      value     => "yes",
    }

### delete entry

    ssh_config { "HashKnownHosts":
      ensure => absent,
    }

    ssh_config { "BatchMode":
      ensure    => absent,
      host      => "example.net",
    }

### manage entry in another ssh_config location

    ssh_config { "CheckHostIP":
      ensure => present,
      value  => "yes",
      target => "/etc/ssh/another_sshd_config",
    }

## sshd_config provider

This is a custom type and provider supplied by `augeasproviders`.

### manage simple entry

    sshd_config { "PermitRootLogin":
      ensure => present,
      value  => "yes",
    }

### manage array entry

    sshd_config { "AllowGroups":
      ensure => present,
      value  => ["sshgroups", "admins"],
    }

### manage entry in a Match block

    sshd_config { "X11Forwarding":
      ensure    => present,
      condition => "Host foo User root",
      value     => "yes",
    }

    sshd_config { "AllowAgentForwarding":
      ensure    => present,
      condition => "Host *.example.net",
      value     => "yes",
    }

### manage entries with same name in different blocks

    sshd_config { "X11Forwarding global":
      ensure => present,
      key    => "X11Forwarding",
      value  => "no",
    }

    sshd_config { "X11Forwarding foo":
      ensure    => present,
      key       => "X11Forwarding",
      condition => "User foo",
      value     => "yes",
    }

    sshd_config { "X11Forwarding root":
      ensure    => present,
      key       => "X11Forwarding",
      condition => "User root",
      value     => "no",
    }

### delete entry

    sshd_config { "PermitRootLogin":
      ensure => absent,
    }

    sshd_config { "AllowAgentForwarding":
      ensure    => absent,
      condition => "Host *.example.net User *",
    }

### manage entry in another sshd_config location

    sshd_config { "PermitRootLogin":
      ensure => present,
      value  => "yes",
      target => "/etc/ssh/another_sshd_config",
    }

## sshd_config_subsystem provider

This is a custom type and provider supplied by `augeasproviders`.

### manage entry

    sshd_config_subsystem { "sftp":
      ensure  => present,
      command => "/usr/lib/openssh/sftp-server",
    }

### delete entry

    sshd_config_subsystem { "sftp":
      ensure => absent,
    }

### manage entry in another sshd_config location

    sshd_config_subsystem { "sftp":
      ensure  => present,
      command => "/usr/lib/openssh/sftp-server",
      target  => "/etc/ssh/another_sshd_config",
    }

## sshkey provider

### manage entry

    sshkey { "foo.example.com":
      ensure  => present,
      type    => "ssh-rsa",
      key     => "AAADEADMEAT",
    }

### manage entry with aliases

    sshkey { "foo.example.com":
      ensure       => present,
      type         => "ssh-rsa",
      key          => "AAADEADMEAT",
      host_aliases => [ 'foo', '192.168.0.1' ],
    }

### manage hashed entry

    sshkey { "foo.example.com":
      ensure        => present,
      type          => "ssh-rsa",
      key           => "AAADEADMEAT",
      hash_hostname => true,
    }

### hash existing entry

    sshkey { "foo.example.com":
      ensure        => hashed,
      type          => "ssh-rsa",
      key           => "AAADEADMEAT",
      hash_hostname => true,
    }

### delete entry

    sshkey { "foo.example.com":
      ensure => absent,
    }

### manage entry in another ssh_known_hosts location

    sshkey { "foo.example.com":
      ensure  => present,
      type    => "ssh-rsa",
      key     => "AAADEADMEAT",
      target  => "/root/.ssh/known_hosts",
    }

## sysctl provider

This is a custom type and provider supplied by `augeasproviders`.

### manage simple entry

    sysctl { "net.ipv4.ip_forward":
      ensure => present,
      value  => "1",
    }

### manage entry with comment

    sysctl { "net.ipv4.ip_forward":
      ensure  => present,
      value   => "1",
      comment => "test",
    }

### delete entry

    sysctl { "kernel.sysrq":
      ensure => absent,
    }

### remove comment from entry

    sysctl { "kernel.sysrq":
      ensure  => present,
      comment => "",
    }

### manage entry in another sysctl.conf location

    sysctl { "net.ipv4.ip_forward":
      ensure => present,
      value  => "1",
      target => "/etc/sysctl.d/forwarding.conf",
    }

### do not update value with the `sysctl` command

    sysctl { "net.ipv4.ip_forward":
      ensure => present,
      value  => "1",
      apply  => false,
    }

## syslog provider

This is a custom type, with two providers supplied by `augeasproviders`.  A
`syslog` provider handles basic syslog configs, while an `rsyslog` provider
handles the extended rsyslog config (this requires Augeas 1.0.0).

### manage entry

    syslog { "my test":
      ensure      => present,
      facility    => "local2",
      level       => "*",
      action_type => "file",
      action      => "/var/log/test.log",
    }

### manage entry with no file sync

    syslog { "cron.*":
      ensure      => present,
      facility    => "cron",
      level       => "*",
      action_type => "file",
      action      => "/var/log/cron",
      no_sync     => true,
    }

### manage remote hostname entry

    syslog { "my test":
      ensure      => present,
      facility    => "local2",
      level       => "*",
      action_type => "hostname",
      action      => "centralserver",
    }

### manage remote hostname entry with port and protocol

    syslog { "my test":
      ensure          => present,
      facility        => "local2",
      level           => "*",
      action_type     => "hostname",
      action_port     => "514",
      action_protocol => "tcp",
      action          => "centralserver",
    }

### manage user destination entry

    syslog { "my test":
      ensure      => present,
      facility    => "local2",
      level       => "*",
      action_type => "user",
      action      => "root",
    }

### manage program entry

    syslog { "my test":
      ensure      => present,
      facility    => "local2",
      level       => "*",
      action_type => "program",
      action      => "/usr/bin/foo",
    }

### delete entry

    syslog { "mail.*":
      ensure      => absent,
      facility    => "mail",
      level       => "*",
      action_type => "file",
      action      => "/var/log/maillog",
    }

### manage entry in rsyslog

    syslog { "my test":
      ensure      => present,
      facility    => "local2",
      level       => "*",
      action_type => "file",
      action      => "/var/log/test.log",
      provider    => "rsyslog",
    }

### manage entry in another syslog location

    syslog { "my test":
      ensure      => present,
      facility    => "local2",
      level       => "*",
      action_type => "file",
      action      => "/var/log/test.log",
      target      => "/etc/mysyslog.conf",
    }
