## What is this?
This app originally saw the light of day to support the [MI Diaries Project](https://mi-diaries.org), and through grant funding from the [National Science Foundation](https://nsf.gov) is being turned into a ready-to-use semi-turnkey solution for (researchers? educators? heathens?) to use in similar projects. While primarily focused on providing an easy to use interface for generating transcripts of the collected data for linguistic analysis, it can be used for a host of other projects, as the code can be modified to enable/disable features as needed or wanted. It's based on open source projects, so it can be freely edited and modified to fit _your_ needs.

The goal of this was to provide both the mobile and admin apps as easy to use, configurable, and deployable versions of the MI Diaries system for use by any researcher or educator, without requiring much IT unit time; once configured, you can work with your team to get the admin portal accessible online, the mobile app deployed, and begin collecting data! (examples? the greek one for 'collecting memories from past archeological digs in Greece, and another providing transcription access and editing for a prior project analyzing Philadelphia English')

The two components of this project, the frontend and backend, provide an admin portal to manage data primarily provided via the [MI Diaries App Builder](https://builder.mi-diaries.org), which, with some basic info and selections, provides the code for a buildable mobile app for Android and iOS based off our current production MI Diaries mobile app.

The admin portal is free-standing as well, and can be utilized without the mobile app; when using without the mobile app, it still gives you a way to manually create subjects, upload audio, process uploaded audio files to generate editable transcripts (including error detection such as overlapping times, missing words, etc). Blah blah maybe remove this block
    

## For production deployment

- Talk to your IT person/unit, and they can configure the app for you -  using the information below - and provide you the URL which you can access the site at to continue along with using the app.
- This step will ultimately be what most use, as the vanilla version of the Admin Portal is ready to use, albeit with settings configured based off of past projects and usages we've seen. Provide them the URL you're currently at, and they can do the rest for you
- While you wait for it to be deployed for your use, go take a look at [MI Diaries App Builder](https://builder.mi-diaries.org) and configure your mobile app however you would like it to look
    * generate and provide the Form Secret to your IT person/unit for use in the steps below
- Relax, and watch your research data roll in once you've deployed the mobile app(s)!

## For IT folks, or local dev/testing

### Requirements (and assumptions)
In order to use, and eventually deploy, the admin portal, you'll (typically) need to have either an IT person/unit that you can rely on to hand this off to for the final hosting, or have access to/on your system: 

- Node
    * the app was written originally with v16, but newer versions may work as well
    * v16 works best for CI builds and expectations currently
- Yarn installed globally
    * you can stick with plain NPM as well but... yarn makes things prettier - and faster!
- access to a MySQL 5.7 database with username and password
    * 5.7 is used currently due to some query limitations and how the data is stored in JSON structures
    * ensures the widest reach for environments that can't afford to redesign their infrastructure, already are running a database setup, and can't (or don't want to) run alternatives such as MariaDB.
    * 8.0 also works (at least on Windows)
    * **on the roadmap**: rewriting the few manual queries that exist to use the framework, which will allow a broader range of database options; testing with postgres and mariadb will follow
- ffmpeg installed
    * note the paths to ffmpeg and ffprobe, as these are both necessary for the backend to function
- Google Cloud Platform (GCP) access
    * the API relies on GCP for two pieces:
        + Cloud Storage API: temporary storage of the audio file for transcription
        + Speech-to-Text API: to create the actual transcription text for the audio file
    * this will require a JSON file containing a service account that has access to the Storage API, referenced in the backend's `.env` file
    * if you don't know what this part is, that's ok! Talk to your IT folks and see about what they can provision for you.
    * **on the roadmap** Don't have any access to Google Cloud Platform at your institution and need transcripts still? We're working on a version that will (hopefully) provide a transcription service that can be swapped out to other providers; however GCP is still the fastest, cheapest, and most performant so far, with transcript times being less than 1/2 realtime of the audio file
- Final production: Docker host for the two containers, or a server that can run the Node Backend server and somewhere to deploy the production build of the Frontend server

## Getting started
The admin portal consists of two portions - a web frontend, and an API backend.
The frontend is written in JavaScript, utilizing [Vue](https://vuejs.org) styled with [Vuetify](https://vuetifyjs.com), and the API is running on top of [FeathersJS](https://feathersjs.com) to provide an easy to use and extensible framework for both database interaction and the handling of the audio files from the mobile app or directly inside the admin portal.


### Cloning projects
First, you'll need to clone down the two repositories:

- frontend: https://github.com/midiaries/admin-frontend
- backend: https://github.com/midiaries/admin-backend

Once cloned, they're able to be configured via simple .env files, which provide the environment variables the apps are expecting - regardless of whether it's running in production inside a Docker container (with an example docker compose file included in each for spinning up a container) or directly on your system (via yarn)

### Configuring environments

Before you start using the admin portal for the first time, you'll need to do some configurations - `example.env` is provided in each repository to make it quick and easy to get moving

In both cloned repositories, copy `example.env` to `.env`, then add in your information:

####Frontend:

```
API_BASE_URL=
VUE_APP_API_BASE_URL=
VIRTUAL_HOST=
TZ=
```

- `API_BASE_URL`: location of API during development running via `yarn dev`
    * example: `http://localhost:3000`
    * will rewrite `http[s]://frontend-example-url/api` to this path, dealing with CORS and other issues for you
- `VUE_APP_API_BASE_URL`: location of API during production (e.g. container name and port specified in backend `.env`
    * example: `http://container-name:3000`
    * will rewrite `http[s]://frontend-example-url/api` to this path, dealing with CORS and other issues for you
- `VIRTUAL_HOST`: used for some reverse proxies to auto-detect and configure
    * example: `frontend-url.localhost`
- `TZ`: time zone the server is running in
    * example: `America/Detroit`

####Backend:

```
APP_BASE=
APP_NAME=
DBHOST=
DBNAME=
DBUSER=
DBPASS=
DBTYPE=mysql
FFMPEG_PATH=/usr/local/bin/ffmpeg
FFPROBE_PATH=/usr/local/bin/ffprobe
FORM_SECRET=
FROM_EMAIL=
GCS_BUCKET_NAME=
GCS_FOLDER_NAME=
GCS_PRIVATE_KEY_PATH=
GCS_USE_FOLDER=true
NODE_ENV=development
PORT=3000
RECAPTCHA_SECRET=
SECRET=
SMTP_HOST=
SMTP_PASS=
SMTP_USER=
SUBJECT_PREFIX=
SUBJECT_LENGTH=
TZ=
VIRTUAL_HOST=
```

- `APP_BASE`: full URL of site
    *  example: `https://app-name.your-url.edu`
- `APP_NAME`: name displayed in headers on site
    *  example: `MI Diaries`
- `DBHOST`: database server - hostname
- `DBNAME`: database server - database name
- `DBUSER`: database server - username for connection
- `DBPASS`: database server - user password for connection
- `DBTYPE`: database server - type, just use `mysql` for now or it'll probably break
- `FFMPEG_PATH`: path on server to `ffmpeg` executable
    * if using the Dockerfile in the repo, will be at `/usr/local/bin/ffmpeg`
- `FFPROBE_PATH=`: path on server to `ffprobe` executable
    * if using the Dockerfile in the repo, will be at `/usr/local/bin/ffprobe`
- `FORM_SECRET`: secret key to use for forms and mobile app to talk to your admin portal
    * can use the [MI Diaries App Builder](https://builder.mi-diaries.org/build) to generate it for you
- `FROM_EMAIL`: email address for sending password reset, account activations, signups, notices, etc
    * example: `App Support<apps@mi-diaries.org>`
- `GCS_BUCKET_NAME`: bucket name on Cloud Storage
    * example: `youractualbucketname`
- `GCS_FOLDER_NAME`: folder name on Cloud Storage, made inside `GCS_BUCKET_NAME` location
    * example: `transcriber_uploads`
- `GCS_PRIVATE_KEY_PATH`: path on server to the GCS service account key
    * if using the Docker compose file in the repo, you can just use `./gcs-key.json` and set up the mount in `docker-compose.yml`
- `GCS_USE_FOLDER`: whether to drop uploads in a folder or not
    * just use `true` unless you have good reason not to!
- `NODE_ENV`: environment the system is running in - `development` or `production` - to pass to the node server
    * enables hot reload and watching files, vs proper production methods
- `PORT`: port the API server will listen on
    * example: `3000`
    * if using the Docker compose file in the repo, you can stick with `3000`
- `RECAPTCHA_SECRET`: Google ReCaptcha key for signup captchas
    * currently unused
- `SECRET`: long random string or phrase (without spaces) to use as an encryption base for authentication tokens
    * example: `ImNotSureWhatYouWantFromThisButHereItIsAnyway`
- `SMTP_HOST`: email server name
    * example: `mail.yourdomain.com`
- `SMTP_PASS`: user pass when accessing `SMTP_HOST`
- `SMTP_USER`: user name when accessing `SMTP_HOST`
- `SUBJECT_PREFIX`: characters to use to identify project/subjects, capitalized
    * example: `MCD`
    * produces IDs in form of `{SUBJECT_PREFIX}-{zero-padded numbering of SUBJECT_LENGTH}`
- `SUBJECT_LENGTH`: number of characters to use for subject number
    * example: `5`
    * examples above would produce `MCD-00001`, `MCD-00002`, etc
- `TZ`: time zone the server is running in
    * example: `America/Detroit`
- `VIRTUAL_HOST`: used for some reverse proxies to auto-detect and configure
    * example: `backend-url.localhost`

### Starting up the application

Assuming you're running this locally on your computer, and have configured `.env` in both cloned repos, you're pretty much ready to start!

###### Start up the backend

- Open a new terminal window and change directory to where you cloned the backend repo, and just configured the `.env` for your setup
- run `yarn install` and wait while it fetches and installs all the dependencies needed
- once completed, run `yarn sequelize db:migrate` to initialize the database with all the migrations
    * this step sets up all the tables, configures some starting defaults, and preps everyting for you
    * while the migrations run, it'll output what it's doing and what step it is on, including outputting the default Username and Password for logging in the first time
- once migrations are completed successfully, you can run `yarn dev` to start the backend
- when successful, the screen should show something similar to:
```
  yarn run v1.22.19
  $ nodemon src/
  [nodemon] 1.19.4
  [nodemon] to restart at any time, enter `rs`
  [nodemon] watching dir(s): *.*
  [nodemon] watching extensions: js,mjs,json
  [nodemon] starting `node src/`
  info: MCD API v1.8.3 started on http://localhost:3002
```
###### Start up the frontend

- Open a new terminal window, and change directory to where you cloned the frontend repo, and just configured the `.env` for your setup
- run `yarn install` and wait while it fetches and installs all the dependencies needed
- once completed, pick a port number (e.g. 8080) that's free on your system, and run `yarn serve --port {that port}`
    * example: `yarn serve --port 8082`
- it'll build everything needed to run the frontend, and once completed, the screen should show something similar to:
```
  DONE  Compiled successfully in 5738ms


  App running at:
  - Local:   http://localhost:8082/
  - Network: http://xxx.yyy.zzz.ttt:8082/

  Note that the development build is not optimized.
  To create a production build, run yarn build.
```

###### Access the site

- Open your browser, and visit the url listed abovea nd add /admin at the end, in this case, http://localhost:8082/admin
- On first setup, you'll need to login with the following credentials:
    * Email Address: `setup@this.app`
    * Password: `ChangeMePlease!`
