# Tiny EC2 Bootstrapper

This is designed to do the minimal amount of work required to bootstrap an EC2
instance based on the local settings assigned at boot time as well as the
user's configured settings. This is in-concept similar to
[cloud-init](https://cloudinit.readthedocs.io/en/latest/) but trades features
and cloud platform support for small size and limited external dependencies.

## Requirements

The most important feature of this bootstrapper is the very limited set of
dependencies. In-fact this works with just busybox provided the wget applet is
built-in. The only required dependencies are:

- bash-like shell (e.g. bash, dash, ash)
- wget
- resize2fs

## Supported Features and Environments

cloud-init has support for many different cloud providers. This project only
supports EC2, specifically the [EC2 metadata
service](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html)
is a hard requirement of using this bootstrapper. All of the data for the
supported features below is sourced from the EC2 instance metadata service
which runs on every EC2 instance at IP 169.254.169.254.

cloud-init also has a very rich feature set with support for adding users,
installing packages, and many other things. This bootstrap does not support
those things. Instead it supports:

- setting system hostname
- install user's configured SSH keys to the alpine user's authorized_keys file
- run any script-like user data (must start with #!)
- disable root and alpine password
- resize root partition to available disk space

These steps only run once. After the initial bootstrap the bootstrapper script
is a no-op. To force the script to run again at boot time remove the file
`/var/lib/cloud/.bootstrap-complete` and reboot the instance.

## User Data

User data is provided at instance boot time and can be any arbitrary string of
data. The bootstrapper will consider any user data that begins with the ASCII
characters '#!' to be a script. It will write the entire contents of the user
data to `/var/lib/cloud/user-data.sh`, make the file executable, and execute
the file piping any output to `/var/log/cloud-bootstrap.log`.

The user data script can do anything it pleases with the instance. It will be
run as root and networking will be up. No other grantees about system state are
made at the point the script runs.

## Assumptions

- This was written for Alpine Linux and thus assumes that the login user is
  called alpine. This could be configurable in the future but currently is not.

- The script is run by OpenRC
