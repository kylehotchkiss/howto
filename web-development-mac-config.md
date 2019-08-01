# Web Development Mac Config
I've been using a special configuration on my Mac computers the past 10 years to help me develop multiple sites that can be quickly bootstrapped without changing any Apache config files or hostname file.

This avoids a few patterns in local site development I dislike, such as:
* Having to edit hostfiles
* Having to edit apache vhost file
* Hosting development projects from within home folder

## Step 1: Configuring your user account, groups, and directories

MacOS has a built in `_www` group that Apache is already configured with. This means any directory owned by `_www` group won't require additional changes for Apache to edit. Your Mac user account _isn't_ a part of `_www` despite being admin as it's intended for internal use.

`$ sudo dseditgroup -o edit -a $USER -t user _www`

Now verify it worked:

`$ dseditgroup -o checkmember -m $USER _www`

You should see this:
`yes %your-username% is a member of _www`

Now, let's create a `/Sites` directory and make sure that both our user and Apache have access to it by running the following commands:

```
$ sudo mkdir /Sites
$ sudo mkdir /Sites/_apache
$ sudo touch /Sites/_apache/error.log /Sites/_apache/access.log
$ sudo chgrp -R _www /Sites
$ sudo chmod -R 775 /Sites
```

## Step 2: Configuring Apache
