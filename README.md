[![build status][251]][232] [![commit][255]][231] [![version:x86_64][256]][235] [![size:x86_64][257]][235] [![version:armhf][258]][236] [![size:armhf][259]][236]

## [Alpine-Akaunting][234]
#### Container for Alpine Linux + PHP7 + Akaunting
---

This [image][233] containerizes [Akaunting][137] accounting webapp
along with its php dependencies to track metrics for a peson or
small group.  **Database not included**. Checkout
[alpine-mysql][139] to configure your own MySQL server in
a container.

Based on [Alpine Linux][131] from [alpine-php][132] image with
[NGINX][135], [PHP7][136] and the [s6][133] init system
[overlayed][134] in it.

**Auto updated according to the [Github][138] release**

Updated with latest php7 packages and proxied out through NGINX,
which can be also used to serve the static files.

The image is tagged respectively for the following architectures,
* **armhf**
* **x86_64**

**armhf** builds have embedded binfmt_misc support and contain the
[qemu-user-static][105] binary that allows for running it also inside
an x64 environment that has it.

---
#### Get the Image
---

Pull the image for your architecture it's already available from
Docker Hub.

```
# make pull
docker pull woahbase/alpine-akaunting:x86_64
```

---
#### Configuration Defaults
---

* Akaunting is located at the endpoint `/akaunting/`, with the
  webapp at `/config/akaunting/`.

* Akaunting source is located at `/opt/akaunting/akaunting.zip`.

* These configurations are inherited from the nginx image:

    * Drop privileges to `alpine` whenever configured to. Respects
      `PUID` / `PGID`.

    * Binds to both http(80) and https(443). Publish whichever you
      need, or both.

    * Default configs setup a static site at `/` by copying
      `/defaults/index.html` at the webroot location
      `/config/www/`.  Mount the `/config/` locally to persist
      modifications (or your webapps). NGINX configs are at
      `/config/nginx`, and vhosts at `/config/nginx/site-confs/`.

    * 4096bit Self-signed SSL certificate is generated in first
      run at `/config/keys`. Pass the runtime variable
      `SSLSUBJECT` with a valid info string to make your own.

    * `.htpasswd` is generated with default credentials
      `admin/insecurebydefault` at `/config/keys/.htpasswd`

    * Sets up a https and auth protected web location at `/secure`.

    * If you're proxying multiple containers at the same host, or
      reverse proxying multiple hosts at the same container, you
      may need to add `--net=host` and/or add entries in your
      firewall to allow traffic.

---
#### Run
---

If you want to run images for other architectures, you will need
to have binfmt support configured for your machine. [**multiarch**][104],
has made it easy for us containing that into a docker container.

```
# make regbinfmt
docker run --rm --privileged multiarch/qemu-user-static:register --reset
```

Without the above, you can still run the image that is made for your
architecture, e.g for an x86_64 machine..

Running `make` starts the service.

```
# make
docker run --rm -it \
  --name docker_akaunting --hostname akaunting \
  -e PGID=1000 -e PUID=1000 \
  -c 256 -m 256m \
  -p 80:80 -p 443:443 \
  -v config:/config \
  -v /etc/hosts:/etc/hosts:ro \
  -v /etc/localtime:/etc/localtime:ro \
  woahbase/alpine-akaunting:x86_64
```

Stop the container with a timeout, (defaults to 2 seconds)

```
# make stop
docker stop -t 2 docker_akaunting
```

Removes the container, (always better to stop it first and `-f`
only when needed most)

```
# make rm
docker rm -f docker_akaunting
```

Restart the container with

```
# make restart
docker restart docker_akaunting
```

---
#### Shell access
---

Get a shell inside a already running container,

```
# make shell
docker exec -it docker_akaunting /bin/bash
```

set user or login as root,

```
# make rshell
docker exec -u root -it docker_akaunting /bin/bash
```

To check logs of a running container in real time

```
# make logs
docker logs -f docker_akaunting
```

---
### Development
---

If you have the repository access, you can clone and
build the image yourself for your own system, and can push after.

---
#### Setup
---

Before you clone the [repo][231], you must have [Git][101], [GNU make][102],
and [Docker][103] setup on the machine.

```
git clone https://github.com/woahbase/alpine-akaunting
cd alpine-akaunting
```
You can always skip installing **make** but you will have to
type the whole docker commands then instead of using the sweet
make targets.

---
#### Build
---

You need to have binfmt_misc configured in your system to be able
to build images for other architectures.

Otherwise to locally build the image for your system.
[`ARCH` defaults to `x86_64`, need to be explicit when building
for other architectures.]

```
# make ARCH=x86_64 build
# sets up binfmt if not x86_64
docker build --rm --compress --force-rm \
  --no-cache=true --pull \
  -f ./Dockerfile_x86_64 \
  --build-arg ARCH=x86_64 \
  --build-arg DOCKERSRC=alpine-php \
  --build-arg PGID=1000 \
  --build-arg PUID=1000 \
  --build-arg USERNAME=woahbase \
  -t woahbase/alpine-akaunting:x86_64 \
  .
```

To check if its working..

```
# make ARCH=x86_64 test
docker run --rm -it \
  --name docker_akaunting --hostname akaunting \
  -e PGID=1000 -e PUID=1000 \
  woahbase/alpine-akaunting:x86_64 \
  sh -ec 'nginx -v; php --version; echo "$(cat /opt/akaunting/version)";'
```

And finally, if you have push access,

```
# make ARCH=x86_64 push
docker push woahbase/alpine-akaunting:x86_64
```

---
### Maintenance
---

Sources at [Github][106]. Built at [Travis-CI.org][107] (armhf / x64 builds). Images at [Docker hub][108]. Metadata at [Microbadger][109].

Maintained by [WOAHBase][204].

[101]: https://git-scm.com
[102]: https://www.gnu.org/software/make/
[103]: https://www.docker.com
[104]: https://hub.docker.com/r/multiarch/qemu-user-static/
[105]: https://github.com/multiarch/qemu-user-static/releases/
[106]: https://github.com/
[107]: https://travis-ci.org/
[108]: https://hub.docker.com/
[109]: https://microbadger.com/

[131]: https://alpinelinux.org/
[132]: https://hub.docker.com/r/woahbase/alpine-php
[133]: https://skarnet.org/software/s6/
[134]: https://github.com/just-containers/s6-overlay
[135]: https://nginx.org
[136]: http://php.net/
[137]: https://akaunting.com
[138]: https://github.com/akaunting/akaunting/releases
[139]: https://hub.docker.com/r/woahbase/alpine-mysql

[201]: https://github.com/woahbase
[202]: https://travis-ci.org/woahbase/
[203]: https://hub.docker.com/u/woahbase
[204]: https://woahbase.online/

[231]: https://github.com/woahbase/alpine-akaunting
[232]: https://travis-ci.org/woahbase/alpine-akaunting
[233]: https://hub.docker.com/r/woahbase/alpine-akaunting
[234]: https://woahbase.online/#/images/alpine-akaunting
[235]: https://microbadger.com/images/woahbase/alpine-akaunting:x86_64
[236]: https://microbadger.com/images/woahbase/alpine-akaunting:armhf

[251]: https://travis-ci.org/woahbase/alpine-akaunting.svg?branch=master

[255]: https://images.microbadger.com/badges/commit/woahbase/alpine-akaunting.svg

[256]: https://images.microbadger.com/badges/version/woahbase/alpine-akaunting:x86_64.svg
[257]: https://images.microbadger.com/badges/image/woahbase/alpine-akaunting:x86_64.svg

[258]: https://images.microbadger.com/badges/version/woahbase/alpine-akaunting:armhf.svg
[259]: https://images.microbadger.com/badges/image/woahbase/alpine-akaunting:armhf.svg
