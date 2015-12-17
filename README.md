freebsd-build-server
=========

Creates a FreeBSD server which provides a ready to run 'poudriere' installation. See [FreeBSD Handbook](https://www.freebsd.org/doc/handbook/ports-poudriere.html) for further information.

You may want to adapt the content of the following files to match your needs:

- `files/port-list`, contains the list of ports you want to build
- `file/poudriere.key`, the key to sign the packages (you may want to replace it with your own or set the value of poudriere_key_file to the path where you have your key stored
- `templates/make.conf.p2`, global build settings used by make
- `templates/poudriere.conf.p2`, poudriere configuration file
- `templates/upload-to-s3`, utility script used to sync packages and package's build option to S3

I wanted to have a build server which I don't have to keep running all the time. To accomplish this, the package repository and the build option for all the packages are synced to S3. This allows me to destroy the build server after the packages have been build and synced. During the next installation the the build options are synced back.

With all that in mind the typical workflow looks like this.

1. Spawn a new server
1. Apply this ansible role
1. Log in
1. Run: `poudriere options -j freebsd-10_2_x64 -p HEAD -f /usr/local/etc/poudriere.d/port-list`
1. Run: `poudriere bulk -j freebsd-10_2_x64 -p HEAD -f /usr/local/etc/poudriere.d/port-list`
1. Run: `upload-to-s3`
1. Log out
1. Destroy the server

Easy as pie.

Requirements
------------

This role is intent to be used with a fresh FreeBSD 10.2 install with some minor modifications. There is a Vagrant Box with providers for VirtualBox and EC2 you may use. I created [this Vagrant project](https://github.com/JoergFiedler/freebsd-build-machine) to create Virtualbox and EC2 machines.

Role Variables
--------------

##### aws_default_region
S3 region to use. Default: `''`.

##### aws_access_key_id
S3 access key. Default: `''`.

##### aws_secret_access_key
S3 secret key. Default: `''`.

##### s3_bucket_name
The bucket to use to store the packages and build options. Default: `''`.

##### s3_upload_path
The path within the S3 bucket where to put `packages` and `build-options` folder. Default: `'/public/FreeBSD'`.

##### freebsd_mirror_server
The FreeBSD mirror server used to set up the jails. Default: `'ftp://ftp.freebsd.org'`.

##### poudriere_ssl_prefix
The path where the package signing key should be saved. Default: `'/usr/local/etc/ssl'`.

##### poudriere_key_file
The private key used to sign the packages. Please change this to use your own key. Default: `'poudriere.key'`.

##### poudriere_jails
The jails which should be created.

    poudriere_jails:
    - { jail_name: 'freebsd-10_2_x64', version: '10.2-RELEASE' }

Default: `''`.

Dependencies
------------

None.

Example Playbook
----------------

    ---
    - hosts: default
      sudo: true

      vars:
        aws_access_key_id: '{{ lookup("env","AWS_ACCESS_KEY_ID") }}'
        aws_secret_access_key: '{{ lookup("env","AWS_SECRET_ACCESS_KEY") }}'
        s3_bucket_name: 'your.fancy.bucket.name'
        poudriere_jails:
        - { jail_name: 'freebsd-10_2_x64', version: '10.2-RELEASE' }

      roles:
      - { role: JoergFiedler.freebsd-build-server }

License
-------

BSD

Author Information
------------------

If you like it or do have ideas to improve this project, please open an issue on Github. Thanks.

