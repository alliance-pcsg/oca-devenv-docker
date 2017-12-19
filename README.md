# Orbis Cascade Alliance primo-explore docker dev environment
[![](https://images.microbadger.com/badges/version/alliance/oca-devenv-docker.svg)](https://microbadger.com/images/alliance/oca-devenv-docker "Get your own version badge on microbadger.com")
[![](https://images.microbadger.com/badges/image/alliance/oca-devenv-docker.svg)](https://microbadger.com/images/alliance/oca-devenv-docker "Get your own image badge on microbadger.com")

This docker image contains the Ex Libris primo-explore dev environment, along with preloaded copies of the Alliance central package and an empty view package. It can be used to rapidly develop primo customizations intended for Alliance use.


## Getting Started

These instructions will help you set up and use the docker image to develop primo customizations.

### Prerequisites

[Docker](https://www.docker.com/docker) and [docker-compose](https://docs.docker.com/compose/install/) are required.

### Setup

Let's assume we're starting development on a new customization and nothing has been written yet. We'll set up a new git repository for the customization and add a compose file:

```sh
$ git init primo-explore-my-customization
$ cd primo-explore-my-customization
$ touch docker-compose.yml
```

We now have a new git repository containing a compose file we can use to manage our dev environment. Let's build out the compose file:

```yaml
version: '3.1'

services:
  devenv:
    image: alliance/oca-devenv-docker:latest
    ports:
      - 8003:8003
    volumes:
      - ./js/:/home/node/primo-explore-devenv/primo-explore/custom/ALLIANCE/js/
```

The `version` key is arbitrary; any version newer than 2 should suffice. Let's examine the `devenv` service, which will start up our dev environment.

The `image` key is pointing to the docker image of our development environment. Docker will automatically download the latest version (`:latest`) if we don't have it when we run compose.

The `ports` key is important - it lets us access the dev environment by mapping our post 8003 to the container's port 8003, on which the Ex Libris dev environment is running.

The `volumes` key is where the magic happens. We are mounting the contents of a local directory called `js/` into the container and using it as the `js/` folder in the `ALLIANCE` view package.

We want to mount at least two files into the view to start with - `bootstrap.js` and `module.js`. These files don't exist yet, so let's create them along with the `js/` folder:

```sh
$ mkdir js && cd js
$ echo "var app = angular.module('viewCustom', ['myCustomization'])" > bootstrap.js
$ echo "angular.module('myCustomization', [])" > module.js
```

We now have declared a new customization module in `module.js` and imported that module to our view in `bootstrap.js`.

### Usage
Let's go back up to the project root and fire up our dev environment.

```sh
$ cd ..
$ docker-compose up
```

You'll see some output as docker pulls the latest version of the image, then the dev environment will start up:
```
Pulling devenv (alliance/oca-devenv-docker:latest)...
latest: Pulling from alliance/oca-devenv-docker
...
Digest: sha256:b65bc76773694178474664f6b38082b541992c075dd796dcaa1fa58bdb8d703c
Status: Downloaded newer image for alliance/oca-devenv-docker:latest
Creating primoexploremycustomization_devenv_1 ...
Creating primoexploremycustomization_devenv_1 ... done
Attaching to primoexploremycustomization_devenv_1
devenv_1  | [18:14:53] Using gulpfile /home/node/primo-explore-devenv/gulpfile.js
...
devenv_1  | [BS] Serving files from: primo-explore
```

Let's open a browser and visit <http://localhost:8003/primo-explore/search?vid=ALLIANCE>. We should see a generic primo-explore view (ALLIANCE). This view will only contain what we add to it in the dev environment.

If we look back at our `js/` directory in the project folder, we'll notice a new file has appeared - `custom.js`. It should look like this:
```js
(function(){
"use strict";
'use strict';

var app = angular.module('viewCustom', ['myCustomization']);

angular.module('myCustomization', []);
})();
```

We don't edit this file, but we can use it to see what the Ex Libris development environment is generating from our provided js files.

Now, we can proceed to develop our customization by adding to our `module.js` file. When we make changes, the dev environment will refresh itself and re-create `custom.js` automatically.

#### The central package
The latest version of the alliance central package is included in the docker image. To test inheritance of new customizations, you can mount a new custom.js into the container's CENTRAL_PACKAGE view using a compose file like below:

```yaml
version: '3.1'

services:
  devenv:
    image: alliance/oca-devenv-docker:latest
    ports:
      - 8003:8003
    volumes:
      - ./js/:/home/node/primo-explore-devenv/primo-explore/custom/ALLIANCE/js/
      - ./central-custom.js:/home/node/primo-explore-devenv/primo-explore/custom/CENTRAL_PACKAGE/js/custom.js
```

You'll need to manually create `central-custom.js`. Editing it will not automatically refresh the dev environment, but you can manually refresh in your browser and the changes should take effect.
