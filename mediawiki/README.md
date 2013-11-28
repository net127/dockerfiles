## MediaWiki

A self-contained instance of MediaWiki with data volume support.

This image contains a basic installation of [MediaWiki][mw], powered by [nginx][nginx] and
[php-fpm][php-fpm]. When first run, the container will not contain a
`LocalSettings.php` and you can run the MediaWiki installer to configure your
wiki:

[mw]: https://www.mediawiki.org/
[nginx]: http://nginx.org/
[php-fpm]: http://php-fpm.org/

    CONFIG_CONTAINER=$(docker run -d nickstenning/mediawiki)

At the end of this process you can place the resulting `LocalSettings.php` in a
directory on the host (say, `/data/wiki`) and restart the container to obtain a
fully-configured MediaWiki installation.

    docker stop $CONFIG_CONTAINER
    docker run -v /data/wiki:/data -d nickstenning/mediawiki

If you already have a `LocalSettings.php` from a previous MediaWiki
installation, you can simply skip the first step.

By default, MediaWiki uploads will also be written to the `images/` directory of
the mounted data volume. This directory will be created if necessary on startup.

### Technical details

For more information, see [the
repository](https://github.com/nickstenning/dockerfiles/tree/master/mediawiki).

=== Setup ===

There's probably a better way, but this is what I'm doing at the moment:

* connect to http://localhost:<PORT> and configure mediawiki
* copy LocalSettings.php to /opt/mediawiki on the docker server
* remove "$wgServer" from LocalSettings.php, if using dynamic ports
* move mysql db to /opt/mediawiki on the docker server:
** use 'docker ps' to get the container id of the container you just setup, then:

```
docker commit <CONTAINER_ID> net127/mediawiki
docker stop <CONTAINER_ID>
docker run -i -t -v /opt/mediawiki:/data net127/mediawiki /bin/bash
mv /var/lib/mysql /data
mv /etc/mysql/my.cnf /etc/mysql/my.cnf.orig
sed 's/^datadir.*/datadir = \/data\/mysql/' </etc/mysql/my.cnf.orig >/etc/mysql/my.cnf
```

(from another shell on the docker server):
```
docker commit <CONTAINER_ID> net127/mediawiki
```

exit container
from net127/dockerfiles/mediawiki:
```
./start
```

connect to http://localhost:<PORT>

...now your data will presist!
