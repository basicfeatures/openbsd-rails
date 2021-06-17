# Ruby On Rails 7.0 on OpenBSD 7.0

### Template for a minimalistic ultra-secure server setup for your Ruby On Rails apps.

> Choose OpenBSD for your Unix needs. OpenBSD -- the world's simplest and most secure Unix-like OS. An alternatve to the vulnerabilities and overengineering of Linux and its related network services such as NGiNX/Apache, OpenSSL, iptables/nftables, systemd, BIND, Postfix and much, much more. OpenBSD -- the cleanest kernel, the cleanest userland and the cleanest configuration syntax.

[`httpd(8)`](https://man.openbsd.org/httpd.8) [(PDF: brief intro)](https://www.openbsd.org/papers/httpd-asiabsdcon2015.pdf) and [`acme-client(8)`](https://man.openbsd.org/acme-client.1) are for [Let's Encrypt](https://letsencrypt.com/) TLS-certificates and their [ACME-challenges](https://letsencrypt.org/docs/challenge-types/). [`relayd(8)`](https://man.openbsd.org/relayd.8) does the reverse proxying and TLS-termination, and behind all that is Ruby On Rails' [Puma](https://puma.io/) webserver, managed by [`rc.subr(8)`](https://man.openbsd.org/rc.subr) system startup scripts. [`nsd(8)`](https://man.openbsd.org/nsd.8) does the DNS, and [`pf(8)`](https://www.openbsd.org/faq/pf/) the firewalling.

## As root

Edit config files to taste, then copy them to root:

    cp -R etc/ /
    cp -R var/ /

Add packages:

    pkg_add zsh ruby postgresql-server node redis sass

Symlinks from Ruby's post-install message:

    ln -s /usr/local/bin/ruby30 /usr/local/bin/ruby
    ln -s /usr/local/bin/erb30 /usr/local/bin/erb
    ln -s /usr/local/bin/irb30 /usr/local/bin/irb
    ln -s /usr/local/bin/rdoc30 /usr/local/bin/rdoc
    ln -s /usr/local/bin/ri30 /usr/local/bin/ri
    ln -s /usr/local/bin/rake30 /usr/local/bin/rake
    ln -s /usr/local/bin/gem30 /usr/local/bin/gem
    ln -s /usr/local/bin/bundle30 /usr/local/bin/bundle
    ln -s /usr/local/bin/bundler30 /usr/local/bin/bundler
    ln -s /usr/local/bin/racc30 /usr/local/bin/racc
    ln -s /usr/local/bin/racc2y30 /usr/local/bin/racc2y
    ln -s /usr/local/bin/y2racc30 /usr/local/bin/y2racc

Nokogiri dependencies:

    pkg_add libiconv libxml libxslt

Create unprivileged user and group `myuser`:

    useradd -m -s /usr/local/bin/zsh myuser
    passwd myuser

## As unprivileged user

Make Bundler install gems locally:

    bundle config set path /home/myuser/.bundle/

Set gem path:

    echo "path+=(/home/myuser/.gem/ruby/3.0/bin)" >> ~/.zprofile
    source ~/.zprofile

Install Rails, and make sure Nokogiri uses OpenBSD's packages when compiling:

    gem install --user-install rails -- --use-system-libraries

## Let's Encrypt HTTPS/TLS

Modify `etc/acme-client.conf`, `etc/relayd.conf` and `etc/httpd.conf` accordingly:

    acme-client -v myapp.com
    acme-client -v www.myapp.com

Check ratings at [SSL Labs](https://ssllabs.com/ssltest/) and [Security Headers](https://securityheaders.com/).

## PostgreSQL

    doas -u _postgresql initdb -D /var/postgresql/data/ -U postgres

    rcctl enable postgresql
    rcctl start postgresql

Create database:

    doas -u _postgresql psql -U postgres
    CREATE ROLE myapp LOGIN SUPERUSER PASSWORD '<your password>';

Encrypt passwords:

    EDITOR=vim rails credentials:edit

    database:
      username: myapp
      password: <your password>

### Start system services

    rcctl enable nsd
    rcctl start nsd

    rcctl enable httpd
    rcctl start httpd

    rcctl enable relayd
    rcctl start relayd

### Start your app

    rcctl enable myapp
    rcctl start myapp

