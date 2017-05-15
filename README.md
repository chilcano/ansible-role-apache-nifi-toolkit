# Ansible Role: apache-nifi-toolkit

An Ansible Role that installs, configures [Apache NiFi Toolkit](https://nifi.apache.org) and runs commands to get X.509 and NiFi configuration files.
The Apache NiFi Toolkit is a tool to facilitate the secure setup of NiFi, in other words, this toolkit generates the NiFi configuration and X.509 Certificates, Java Key Store and Java Trust Store required for a secure configuration of NiFi. Further information can be found here: https://nifi.apache.org/docs/nifi-docs/html/administration-guide.html#tls-generation-toolkit

This Role provides the following features:

- Installs Apache NiFi Toolkit.
- Performs Apache NiFi Toolkit commands. The `standalone` is the unique NiFi TLS Toolkit Service implemented.
- Synchronizes (remote to local host) and keeps in an unique place all generated NiFi's configuration, X.509 Certificates and Java KeyStore and Java Trust Store files.

## Requirements

- Java.

## Role Variables

Default variables is in `defaults/main.yml`.

## Dependencies

This Ansible Role has not dependencies, although NiFi Toolkit requires Java.
To install Java I will use the (geerlingguy.java)[https://github.com/geerlingguy/ansible-role-java] role but with a few changes to install `Oracle Java 8` in Debian. You have these changes available from my fork at (chilcano.java / branch oracle-java-debian)[https://github.com/chilcano/ansible-role-java/tree/oracle-java-debian]. Just clone that branch.

But if you are going to use the Apache NiFi Toolkit in Ubuntu or CentOS, you will not need the above changes. Just download the `geerlingguy.java` as below, or install manually Java 7 or 8 in your box:

```
$ sudo ansible-galaxy install geerlingguy.java
```

## Example Playbook

```
---
- hosts: nftk1
  become: yes
  vars_files:
    - vars.yml
  roles:
    - role: ../../playbooks/roles/ansible-role-java
      java_packages:
        - oracle-java8-installer
        - ca-certificates
        - oracle-java8-set-default
      java_cleanup: false
      java_home: "/usr/lib/jvm/java-8-oracle"

    - role: chilcano.apache-nifi-toolkit
      nftk:
        version: "1.2.0"
        packaging_bin: "tar.gz"
        packaging_src: "zip"
        action:
          clean:
            installation: false
            repository: false
            dependencies: false
          install: true
          run_cmd: true
          sync: true
        download:
          http_uri: "http://mirror.ox.ac.uk/sites/rsync.apache.org"
      nftk_run:
        cmd: "standalone -n 'nf1.intix.info' --nifiDnSuffix ',OU=INTIX' -C 'CN=chilcano, OU=INTIX' -O -c 'nftk1_ca'"
        clientpasswd: "demo00a"
        keypasswd: "demo00b"
        truststorepasswd: "demo00c"
        keystorepasswd: "demo00d"
      nftk_cfg.dir_repo: "nftk_repo"
      nftk_sync_dir_local: "/Users/Chilcano/1github-repo/binaries"
```


The `inventory` file contains:
```
[nifitks]
nftk1

nftk1 ansible_host=192.168.77.4

[nifitks:vars]
ansible_user=vagrant
ansible_ssh_private_key_file="/Users/Chilcano/.vagrant.d/insecure_private_key"
```

This Ansible Playbook will install Oracle Java, install Apache NiFi Toolkit and perform this command `$ ./bin/tls-toolkit.sh standalone -n 'nf1.intix.info' --nifiDnSuffix ',OU=INTIX' -C 'CN=chilcano, OU=INTIX' -O -c 'nftk1_ca'`. All Keys, Certs and Configs will be created in `$NIFI_TOOLKIT_HOME/nftk_repo` and will be synced to `localhost` under this directory `/Users/Chilcano/1github-repo/binaries`.
The `/Users/Chilcano/1github-repo/binaries` directory needs to be created before. Being synced allow us to get easily and use all files required to configure NiFi securely, one or many times, standalone or cluster.

The final folder structure with Keys, Certificates and configuration files will be as shown below:

![Apache NiFi Toolkit - folder structure and files generated](https://github.com/chilcano/ansible-role-apache-nifi-toolkit/blob/master/ansible-role-apache-nifi-toolkit-folder-struct.png "Apache NiFi Toolkit - folder structure and files")

## License

MIT / BSD

## Author Information

This role was created in 2017 by [Roger Carhuatocto](https://www.linkedin.com/in/rcarhuatocto), author of [HolisticSecurity.io Blog](https://holisticsecurity.io).
