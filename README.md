# Mozilla HTTP Observatory

The Mozilla HTTP Observatory is a set of tools to analyze your website and inform you if you are utilizing the many available methods to secure it.

[![Build Status](https://travis-ci.org/marumari/http-observatory.svg?branch=master)](https://travis-ci.org/marumari/http-observatory) [![Requirements Status](https://requires.io/github/mozilla/http-observatory/requirements.svg?branch=master)](https://requires.io/github/mozilla/http-observatory/requirements/?branch=master)

## Getting started with the HTTP Observatory

First, install the client:
```bash
$ pip install requests
$ git clone https://github.com/mozilla/http-observatory
$ ln -s `pwd`/http-observatory/httpobs/scripts/httpobs /usr/local/bin/httpobs
```

And then scan websites to your heart's content, using our hosted service:

```
$ httpobs www.mozilla.org
Score: 30 [E]
Modifiers:
    [  -5] Initial redirection from http to https is to a different host, preventing HSTS
    [  -5] Subresource Integrity (SRI) not implemented, but all external scripts are loaded over https
    [  -5] X-Content-Type-Options header not implemented
    [ -10] X-XSS-Protection header not implemented
    [ -20] HTTP Strict Transport Security (HSTS) header not implemented
    [ -25] Content Security Policy (CSP) header not implemented

$ httpobs www.google.com
Score: 35 [D-]
Modifiers:
    [  +5] Preloaded via the HTTP Public Key Pinning (HPKP) preloading process
    [  -5] X-Content-Type-Options header not implemented
    [ -20] Cookies set without using the Secure flag or set over http
    [ -20] HTTP Strict Transport Security (HSTS) header not implemented
    [ -25] Content Security Policy (CSP) header not implemented

$ httpobs --zero github.com
Score: 106 [A+]
Modifiers:
    [  +5] Preloaded via the HTTP Strict Transport Security (HSTS) preloading process
    [  +5] Subresource Integrity (SRI) is implemented and all scripts are loaded from a similar origin
    [  +1] HTTP Public Key Pinning (HPKP) header set to less than 15 days (1296000)
    [   0] All cookies use the Secure flag and all session cookies use the HttpOnly flag
    [   0] Content is not visible via cross-origin resource sharing (CORS) files or headers
    [   0] Contribute.json isn't required on websites that don't belong to Mozilla
    [   0] Initial redirection is to https on same host, final destination is https
    [   0] X-Content-Type-Options header set to "nosniff"
    [   0] X-Frame-Options (XFO) header set to SAMEORIGIN or DENY
    [   0] X-XSS-Protection header set to "1; mode=block"
    [  -5] Content Security Policy (CSP) implemented with unsafe-inline inside style-src directive
```

If you want additional options, such as to see the raw scan output, use `httpobs --help`:

```
$ httpobs --help
usage: httpobs [options] host

positional arguments:
  host           hostname of the website to scan

optional arguments:
  -h, --help     show this help message and exit
  -d, --debug    output only raw JSON from scan and tests
  -r, --rescan   initiate a rescan instead of showing recent scan results
  -v, --verbose  display progress indicator
  -x, --hidden   don't list scan in the recent scan results
  -z, --zero     show test results that don't affect the final score
```

## Contributing

### Prerequisites
* Python 3
* Git

### Running a local scanner with Docker
* Install [Docker Toolbox](https://www.docker.com/products/docker-toolbox) and [VirtualBox](https://www.virtualbox.org/wiki/Downloads)

```bash
# Install the HTTP Observatory client and requests library
$ git clone https://github.com/mozilla/http-observatory.git
$ cd http-observatory
$ pip3 install .
$ pip3 install --upgrade requests

# Create docker machine
$ docker-machine create --driver virtualbox --virtualbox-disk-size "40000" http-observatory

# Save the URL to the API in your .profile, .bash_profile, or whatever
$ echo export HTTPOBS_API_URL=http://$(docker-machine ip http-observatory):57001/api/v1 >> ~/.profile
$ . ~/.profile

# Start up the docker instance and install all the pieces
$ eval $(docker-machine env http-observatory)
$ docker-compose up -d
```

### Creating a local installation (tested on Ubuntu 15)
```
# Install git, postgresql, and redis
# sudo -s
# apt-get install -y git libpq-dev postgresql redis-server

# Clone the repo
# cd /opt
# apt-get clone https://github.com/mozilla/http-observatory.git
# cd http-observatory

# Install the observatory
# pip install .
# pip3 install -r requirements.txt

# Install the database
# su - postgres
$ createdb http_observatory
$ psql http_observatory < httpobs/database/schema.sql
$ psql http_observatory
http_observatory=# \password httpobsapi
http_observatory=# \password httpobsscanner
# vi /etc/postgresql/9.4/main/postgresql.conf (set max_connections = 512, shared_buffers = 256MB)
# service postgresql restart

# Create the httpobs user, and log/pid directories
# useradd -m httpobs
# install -m 750 -o httpobs -g httpobs -d /var/run/httpobs /var/log/httpobs

# Update the environmental variables
# su - httpobs
$ echo export HTTPOBS_API_URL="http://localhost:57001/api/v1" >> ~/.profile

# Start the scanner
$ cd /opt/http-observatory
$ HTTPOBS_DATABASE_USER="httpobsscanner" HTTPOBS_DATABASE_PASS="....." \
    /opt/http-observatory/httpobs/scripts/httpobs-scan-worker

# Start the API (in another terminal)
# HTTPOBS_DATABASE_USER="httpobsapi" HTTPOBS_DATABASE_PASS="....." \
    uwsgi --http :57001 --wsgi-file httpobs/website/main.py --processes 8 --callable app --master


## Authors

* April King

## License

* Mozilla Public License Version 2.0
