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

We're going to use the command line editor `nano` to make some changes to the system Apache install. MacOS suprisingly ships a capable instance of Apache out of the box, and we just need to turn on `vhosts` and `php`. Line numbers in this apache config file tend to change throughout system releases, so use `nano`s `control-w` command to search for the specific line you need to edit. Cancel your search with `control-c` to perform another search

1) Open the apache file for editing
`$ sudo nano /etc/apache2/http.conf`

2) Turn on the Vhosts Alias module
* Search for `LoadModule vhost_alias_module`
* Uncomment this line

3) Turn on the Rewrite module
* Search for `LoadModule rewrite_module`
* Uncomment this line

4) Turn on PHP
* Search for `LoadModule php7_module`
* Uncomment this line

5) Turn on PHP Directory Indexes
* Search for `DirectoryIndex index.html`
* Add in `index.php` like this `DirectoryIndex index.php index.html`

6) Turn on Vhosts config file
* Search for `httpd-vhosts.conf` 
* Uncomment the `#` before `Include`

7) Test out configuration
* Save the file in Nano with `control-o`, then press `return`
* Run `$ apachectl configtest` to verify your changes were valid.

8) Add the vhosts configuration to use directories in /Sites
* `# sudo nano /etc/apache2/extra/httpd-vhosts.conf`
* Replace with the following (put your own values in line 1 and 2):

```
ServerName %your-host-name%
ServerAdmin %your-email-address%

<VirtualHost *:80>
  # Use the Host: header.  
  UseCanonicalName off

  # The location of the sites
  VirtualDocumentRoot /Sites/%0/
  
  # Properly secure the filesystem.  
  <Directory />
    Options FollowSymLinks    
    AllowOverride None
  </Directory>
  
  # Allow access to the sites.  
  <Directory /Sites/>
   Options Indexes FollowSymLinks MultiViews   
   AllowOverride All
   Require all granted  
  </Directory>

  # Log whatever happens.
  ErrorLog /Sites/_apache/error.log  LogLevel warn
  CustomLog /Sites/_apache/access.log combined
</VirtualHost>
```
* Save with `control-o` and `return`

* Run `$ apachectl configtest` again to verify your changes were valid.
* Run `$ sudo apachectl restart` to reboot apache.

# Step 3: Configuring DNSMasq

For this next step, we are going to install software from Homebrew (dnsmasq) which will create a local TLD (I use `.mac` but you can change it to anything that isn't being used as an actual TLD on the internet. Beware the `.dev` is a real TLD with HSTS and is not supported by this setup as of 2018.)

1) Install Homebrew
`$ brew install dnsmasq`

2) Copy config example
`$ cp $(brew list dnsmasq | grep /dnsmasq.conf.example$) /usr/local/etc/dnsmasq.conf`

3) Start DNSMasq on startup
`$ sudo cp $(brew list dnsmasq | grep /homebrew.mxcl.dnsmasq.plist$) /Library/LaunchDaemons/`

4) Handle all .mac TLDs. Change `mac` to your own TLD here if you want to use something different.
`$ echo "address=/mac/127.0.0.1" >> /usr/local/etc/dnsmasq.conf`

5) Start DNSMasq
` $sudo launchctl load /Library/LaunchDaemons/homebrew.mxcl.dnsmasq.plist`

6) Tell OSX to send all .mac requests to DNSMasq. Change `mac` to your own TLD here if you want to use something different.
`$ sudo mkdir -p /etc/resolver`
`$ sudo tee /etc/resolver/mac >/dev/null <<EOF`
`nameserver 127.0.0.1`
`EOF`

# Ping to test
`$ ping your-laptop.mac`
