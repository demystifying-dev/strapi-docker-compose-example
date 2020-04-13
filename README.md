# strapi docker compose example

## Video

[![strapi docker compose example](http://img.youtube.com/vi/-8zJdKypCdQ/0.jpg)](https://youtu.be/-8zJdKypCdQ)

## Ref

- [Strapi installation docs. Installing using Docker](https://strapi.io/documentation/3.0.0-beta.x/installation/docker.html)
  - In Step 1, we select the MongoDB tab
- [Docker Compose file version 3 reference](https://docs.docker.com/compose/compose-file/)

## Install

- Clone this repo, take a look at the `docker-compose.yml` file
- Copy `.env.example` to `.env` and come up with some creative passwords
- Step 2 in the docs:

```bash
% docker-compose pull
Pulling strapiexample ... done
Pulling mongoexample  ... done
victorkane@Victors-MacBook-Air strapi-docker-composer-example % docker-compose up -d
Creating network "strapi-docker-composer-example_strapi-app-network" with driver "bridge"
Creating volume "strapi-docker-composer-example_strapidata" with default driver
Creating strapiexample ... done
Creating mongoexample  ... done
```

Well, it says done, but Strapi is getting your backend api machine ready, so a lot is churning away in the two docker containers we've just set in motion. Depending on your workstation, it might take about 5 minutes before you can point your browser at

- http://localhost:1337/admin

unless, like me, you're using docker-machine, in which case you need to find out the ip first:

```bash
% docker-machine ip default
192.168.88.888
```

and then point your browser at that ip instead of `localhost`

- http://192.168.88.888:1337/admin)

## How can we tell if it's ready?

Let's `docker exec` into the strapi container and see what's going on:

```bash
% docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                      NAMES
7dfdb8c300ed        mongo               "docker-entrypoint.s…"   About a minute ago   Up About a minute   0.0.0.0:27017->27017/tcp   mongoexample
c56195eeddf2        strapi/strapi       "docker-entrypoint.s…"   About a minute ago   Up 39 seconds       0.0.0.0:1337->1337/tcp     strapiexample

docker exec -it strapiexample /bin/bash
root@19480a9d8319:/srv/app# ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 15:50 ?        00:00:00 /bin/sh /usr/local/bin/docker-entrypoint.sh strapi develop
root        13     1  0 15:50 ?        00:00:01 node /usr/local/bin/strapi new . --dbclient=mongo --dbhost=mongo --dbport=27017 --dbname=strapi --dbuserna
root        36    13 88 15:51 ?        00:04:12 node /usr/local/bin/yarnpkg install --production --no-optional
root        47     0  2 15:55 pts/0    00:00:00 /bin/bash
root        52    47  0 15:55 pts/0    00:00:00 ps -ef
```

So `strapi new` is still going on, and `strapi build` hasn't even started yet.

```bash
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 15:50 ?        00:00:02 node /usr/local/bin/strapi develop
root        47     0  0 15:55 pts/0    00:00:00 /bin/bash
root        83    47  0 16:00 pts/0    00:00:00 ps -ef
```

So `strapi new` has finished.

```bash
root@19480a9d8319:/srv/app# ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 15:50 ?        00:00:04 node /usr/local/bin/strapi develop
root        47     0  0 15:55 pts/0    00:00:00 /bin/bash
root        88     1  0 16:00 ?        00:00:00 /bin/sh -c npm run -s build -- --no-optimization
root        89    88 68 16:00 ?        00:00:00 npm
root        96    47  0 16:00 pts/0    00:00:00 ps -ef
```

And now `build` is underway, so it won't be long now.

```bash
root@19480a9d8319:/srv/app# ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 15:50 ?        00:00:05 node /usr/local/bin/strapi develop
root        47     0  0 15:55 pts/0    00:00:00 /bin/bash
root       113     1 56 16:02 ?        00:00:07 /usr/local/bin/node /usr/local/bin/strapi develop
root       124    47  0 16:02 pts/0    00:00:00 ps -ef
```

Great! `build` has cooked up a spanking new Admin UI, we can go ahead and point our browsers there right now!

## What about a log file

Yes we can!

```bash
% docker-compose logs --tail=all -f | grep strapiexample

Attaching to mongoexample, strapiexample
strapiexample    | Using strapi 3.0.0-beta.19.5
strapiexample    | No project found at /srv/app. Creating a new strapi project
strapiexample    | Creating a new Strapi application at /srv/app.
strapiexample    |
strapiexample    | Creating a project from the database CLI arguments.
strapiexample    | Creating files.
strapiexample    | - Installing dependencies:
strapiexample    | Dependencies installed successfully.
strapiexample    |
strapiexample    | Your application was created at /srv/app.
strapiexample    |
strapiexample    | Available commands in your project:
strapiexample    |
strapiexample    |   yarn develop
strapiexample    |   Start Strapi in watch mode.
strapiexample    |
strapiexample    |   yarn start
strapiexample    |   Start Strapi without watch mode.
strapiexample    |
strapiexample    |   yarn build
strapiexample    |   Build Strapi admin panel.
strapiexample    |
strapiexample    |   yarn strapi
strapiexample    |   Display all available commands.
strapiexample    |
strapiexample    | You can start by doing:
strapiexample    |
strapiexample    |   cd /srv/app
strapiexample    |   yarn develop
strapiexample    |
strapiexample    | Starting your app...
strapiexample    | Building your admin UI with development configuration ...
strapiexample    | ℹ Compiling Webpack
strapiexample    | ✔ Webpack: Compiled successfully in 1.18m
strapiexample    |
strapiexample    |  Project information
strapiexample    |
strapiexample    | ┌────────────────────┬──────────────────────────────────────────────────┐
strapiexample    | │ Time               │ Mon Apr 06 2020 21:57:54 GMT+0000 (Coordinated … │
strapiexample    | │ Launched in        │ 10378 ms                                         │
strapiexample    | │ Environment        │ development                                      │
strapiexample    | │ Process PID        │ 97                                               │
strapiexample    | │ Version            │ 3.0.0-beta.19.5 (node v12.13.0)                  │
strapiexample    | └────────────────────┴──────────────────────────────────────────────────┘
strapiexample    |
strapiexample    |  Actions available
strapiexample    |
strapiexample    | One more thing...
strapiexample    | Create your first administrator 💻 by going to the administration panel at:
strapiexample    |
strapiexample    | ┌─────────────────────────────┐
strapiexample    | │ http://localhost:1337/admin │
strapiexample    | └─────────────────────────────┘
strapiexample    |
strapiexample    | [2020-04-06T21:57:54.706Z] debug HEAD /admin (30 ms) 200
strapiexample    | [2020-04-06T21:57:54.726Z] info ⏳ Opening the admin panel...
strapiexample    | [2020-04-06T21:57:54.748Z] info File created: /srv/app/extensions/users-permissions/config/jwt.json
strapiexample    | [2020-04-06T21:58:08.248Z] debug GET /admin (21 ms) 200
strapiexample    | [2020-04-06T21:58:08.545Z] debug GET runtime~main.94ab5055.js (171 ms) 200
strapiexample    | [2020-04-06T21:58:08.552Z] debug GET main.e9e3876e.chunk.js (115 ms) 200
strapiexample    | [2020-04-06T21:58:09.712Z] debug GET /favicon.ico (4 ms) 200
strapiexample    | [2020-04-06T21:58:10.504Z] debug GET /users-permissions/init (38 ms) 401
strapiexample    | [2020-04-06T21:58:10.620Z] debug GET /users/me (35 ms) 401
strapiexample    | [2020-04-06T21:58:10.668Z] debug GET login (31 ms) 200
strapiexample    | [2020-04-06T21:58:12.277Z] debug GET /users-permissions/init (58 ms) 200
strapiexample    | [2020-04-06T21:58:12.471Z] debug GET /admin/init (33 ms) 200
strapiexample    | [2020-04-06T21:58:12.708Z] debug GET 2ff0049a00e47b56bffc059daf9be78b.png (92 ms) 200
strapiexample    | [2020-04-06T21:58:12.711Z] debug GET 6301a48360d263198461152504dcd42b.svg (26 ms) 200
strapiexample    | [2020-04-06T21:58:12.738Z] debug GET cccb897485813c7c256901dbca54ecf2.woff2 (24 ms) 200
strapiexample    | [2020-04-06T21:58:13.135Z] debug GET bd03a2cc277bbbc338d464e679fe9942.woff2 (391 ms) 200
strapiexample    | [2020-04-06T21:58:13.189Z] debug GET 8b4f872c5de19974857328d06d3fe48f.woff2 (39 ms) 200
strapiexample    | [2020-04-06T21:58:13.249Z] debug GET 33d5f0d956f3fc30bc51f81047a2c47d.woff2 (58 ms) 200
strapiexample    | [2020-04-06T21:58:13.263Z] debug GET b15db15f746f29ffa02638cb455b8ec0.woff2 (52 ms) 200
strapiexample    | [2020-04-06T21:58:13.366Z] debug GET 6301a48360d263198461152504dcd42b.svg (101 ms) 200
```

## First Content Type

As usual, create admin user, then create an `article` content type, for example.

## First Article content item

As usual. Let's go into the mongo container and see if it's real!

```bash
% docker exec -it mongoexample /bin/bash
root@7afecba98c09:/# mongo strapi -u strapi -p --authenticationDatabase admin
MongoDB shell version v4.2.5
Enter password:
connecting to: mongodb://127.0.0.1:27017/strapi?authSource=admin&compressors=disabled&gssapiServiceName=mongodb

> show collections
articles
core_store
strapi_administrator
strapi_webhooks
upload_file
users-permissions_permission
users-permissions_role
users-permissions_user
> db.articles.find()
{ "_id" : ObjectId("5e8b3b26845382009a59ffa7"), "title" : "First article", "body" : "## Oh yes\n\nA first article!", "published" : "2020-04-07", "createdAt" : ISODate("2020-04-06T14:22:30.063Z"), "updatedAt" : ISODate("2020-04-06T14:22:30.063Z"), "__v" : 0 }
>
```

## Destroy all our work

This stops the app (both containers), deletes both containers, deletes all the images, and deletes the volume (our data!) as if nothing has happened here:

    % docker-compose down --rmi all -v

If we just want to stop the containers without destroying them and be able to start them again later, in our project root, we do:

```bash
% docker-compose stop
Stopping mongoexample  ... done
Stopping strapiexample ... done
```

Then, to get back to work:

```bash
% docker-compose start
Starting strapiexample ... done
Starting mongoexample  ... done
% docker-compose ps
 Name               Command               State            Ports
--------------------------------------------------------------------------
mongo    docker-entrypoint.sh mongod      Up      0.0.0.0:27017->27017/tcp
strapi   docker-entrypoint.sh strap ...   Up      0.0.0.0:1337->1337/tcp
```

Access the log again (`start` is much quicker than the initial `up` command):

```bash
% docker-compose logs --tail=all -f | grep strapiexample

Attaching to mongoexample, strapiexample
strapiexample    | Starting your app...
strapiexample    |
strapiexample    |  Project information
strapiexample    |
strapiexample    | ┌────────────────────┬──────────────────────────────────────────────────┐
strapiexample    | │ Time               │ Mon Apr 06 2020 22:45:46 GMT+0000 (Coordinated … │
strapiexample    | │ Launched in        │ 10041 ms                                         │
strapiexample    | │ Environment        │ development                                      │
strapiexample    | │ Process PID        │ 16                                               │
strapiexample    | │ Version            │ 3.0.0-beta.19.5 (node v12.13.0)                  │
strapiexample    | └────────────────────┴──────────────────────────────────────────────────┘
strapiexample    |
strapiexample    |  Actions available
strapiexample    |
strapiexample    | Welcome back!
strapiexample    | To manage your project 🚀, go to the administration panel at:
strapiexample    | http://localhost:1337/admin
strapiexample    |
strapiexample    | To access the server ⚡️, go to:
strapiexample    | http://localhost:1337
strapiexample    |
strapiexample    | [2020-04-06T22:45:51.371Z] debug GET admin (46 ms) 200
strapiexample    | [2020-04-06T22:45:51.613Z] debug GET runtime~main.94ab5055.js (162 ms) 200
strapiexample    | [2020-04-06T22:45:51.620Z] debug GET main.e9e3876e.chunk.js (32 ms) 200
strapiexample    | [2020-04-06T22:45:52.743Z] debug GET /favicon.ico (4 ms) 200
strapiexample    | [2020-04-06T22:45:53.627Z] debug GET /users-permissions/init (183 ms) 200
strapiexample    | [2020-04-06T22:45:53.682Z] debug GET /admin/init (36 ms) 200
strapiexample    | [2020-04-06T22:45:53.882Z] debug GET /content-manager/content-types (105 ms) 200
strapiexample    | [2020-04-06T22:45:54.371Z] debug GET fb30313e62e3a932b32e5e1eef0f2ed6.png (200 ms) 200
strapiexample    | [2020-04-06T22:45:54.413Z] debug GET 6a7a177d2278b1a672058817a92c2870.png (81 ms) 200
strapiexample    | [2020-04-06T22:45:54.478Z] debug GET fd508c879644dd6827313d801776d714.png (104 ms) 200
strapiexample    | [2020-04-06T22:45:54.533Z] debug GET 47ffa3adcd8b08b2ea82d3ed6c448f31.png (112 ms) 200
strapiexample    | [2020-04-06T22:45:54.625Z] debug GET 75a60ac7a1a65ca23365ddab1d9da73e.png (137 ms) 200
strapiexample    | [2020-04-06T22:45:54.672Z] debug GET a4a1722f025b53b089ca2f6408abf0b7.png (133 ms) 200
strapiexample    | [2020-04-06T22:45:54.685Z] debug GET d6d70dd7cb470ff10ecc619493ae7205.png (95 ms) 200
strapiexample    | [2020-04-06T22:46:17.213Z] debug GET /content-type-builder/components (28 ms) 200
strapiexample    | [2020-04-06T22:46:17.257Z] debug GET /content-type-builder/content-types (40 ms) 200
strapiexample    | [2020-04-06T22:46:17.368Z] debug GET 3.79d3ee8f.chunk.js (34 ms) 200
strapiexample    | [2020-04-06T22:46:17.625Z] debug GET 4eb103b4d12be57cb1d040ed5e162e9d.woff2 (52 ms) 200
strapiexample    | [2020-04-06T22:47:05.883Z] debug POST /content-type-builder/content-types (380 ms) 201
strapiexample    | [2020-04-06T22:47:05.895Z] info The server is restarting
strapiexample    |
strapiexample    |
```

Give things a minute to get back into shape, then a refresh on the browser will have us back to work again. And our content type and first article is still there!

This will survive a reboot! So no worries.

## Using the Strapi API

- Go to Roles and Permissions, enable Public permissions for `find` and `find one` for content type Article
- In another browser tab, check out http://192.168.99.101:1337/articles
- Grab an id, and do http://192.168.99.101:1337/articles/5e8b60b30a772e00b351ce7f

## Destroying the containers, images but not the volume!

We can even destroy the images if we wanted to, and our data will not go away, so long as we don't destroy the volume (leave our data):

```bash
% docker-compose down --rmi all
```

Enjoy!

## TODO

- How to use the API instead of the interactive Admin UI for simple CRUD operations
- How to setup docker-compose in this way for a project that is already underway
- Developoment workflow examples (dev, staging, production workflows)
