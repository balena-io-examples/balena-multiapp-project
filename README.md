# Balena multi-app sample

This project demonstrates how to deploy an app that combines multiple existing balena apps into a single `docker-compose.yml` file.

We want to build an application that has the following features:
- Bluetooth audio capabilities (as seen in the `bluetooth-audio` container on the [balenaSound](https://github.com/balena-io-projects/balena-sound/) project)
- Turn device into a hotspot to configure WiFi if no active connection (as seen in the [wifi-connect](https://github.com/balena-io/wifi-connect) project)
- Simple nodejs server (as seen in the [simple-server-node](https://github.com/balena-io-projects/simple-server-node) project)

We can achieve this quite easily in two steps:
1) Get code from existing repositories
2) Build the docker-compose file

## 1. Get code from existing repositories
By using git submodules we can pull code from multiple repositories into our main repo. If you are not familiar with git submodules, please check out the section below.

From our project's root path we do:

````bash
git submodule add https://github.com/balena-io-projects/balena-sound/
git submodule add https://github.com/balena-io/wifi-connect
git submodule add https://github.com/balena-io-projects/simple-server-node
````

This will pull the code from our dependant repositories and link our project to the specific commit we want (current on master branch by default).

## 2. Build the docker-compose file
With the code already in place, we only need to create a docker-compose file that ties everything together. Note that we don't need to pull containers from apps if we don't need them. For example, we only need `bluetooth-audio` from `balenaSound`.

```yaml
version: '2'

services:

  simple-server-node:
    build: ./simple-server-node
    ports:
      - 8080:80
    restart: always

  wifi-connect:
    build: ./wifi-connect
    network_mode: host
    privileged: true
    restart: always
    labels:
      io.balena.features.dbus: '1'

  bluetooth-audio:
    build: ./balena-sound/bluetooth-audio
    network_mode: host
    restart: always
    privileged: true
    labels:
      io.balena.features.dbus: 1
    volumes:
      - bluetoothcache:/var/cache/bluetooth

volumes:
  bluetoothcache:

```



## Git submodules

>Git allows you to include other Git repositories called submodules into a repository. This allows you to track changes in several repositories via a central one. Submodules are Git repositories nested inside a parent Git repository at a specific path in the parent repository’s working directory. A submodule can be located anywhere in a parent Git repository’s working directory and is configured via a .gitmodules file located at the root of the parent repository. This file contains which paths are submodules and what URL should be used when cloning and fetching for that submodule. Submodule support includes support for adding, updating, synchronizing, and cloning submodules.
Git allows you to commit, pull and push to these repositories independently.
Submodules allow you to keep projects in separate repositories but still be able to reference them as folders in the working directory of other repositories.

For more information check out this links:
- https://www.vogella.com/tutorials/GitSubmodules/article.html
- https://git-scm.com/book/en/v2/Git-Tools-Submodules
