# Installation

- Setup the project on Firebase
- Checkout the code and install dependencies
- Setting up your local config 
- Running it to test it out
- Deploying somewhere others can see it
- Changing the domain

## Setup the project on Firebase

### Create the project

Sign into google firebase, so you're on the dashboard page, at https://console.firebase.google.com/

Hit 'add project', giving it a name (referred to as YOUR_PROJECT_NAME from here onwards) , and decide the region/country for this app.

### Naming the project

Once you have decided a name for the project, you'll need to update it in a few places, so that password reset emails appear to come from the correct addresses, and the project name is the one you have chosen, and your users will recognise rather than an autogenerated project number. Update the *public facing name*, at the general settings url:

https://console.firebase.google.com/project/YOUR_PROJECT_NAME/settings/general/

You may also want to update the password reset email text, at:

https://console.firebase.google.com/project/YOUR_PROJECT_NAME/authentication/emails

#### Making sure people can sign in

Finally, you'll want to allow people sign in with email, before you can proceed. Make sure Email/Password is set to `enabled` among the sign-in providers.

https://console.firebase.google.com/project/YOUR_PROJECT_NAME/authentication/providers

Remember, to make sure the password reset email message is correct here.

Congrats! First part sorted!

## Checkout the code and install dependencies

### Clone the project code

You need to download the code and necessary software libraries:


```
git clone git@github.com:productscience/constellate.git
```

### Install the dependencies

Once you have downloaded the main project `cd` into the web directory, and install the dependencies:

``` bash
cd cl8-web
npm install
```

Once you have this, you'll be able to run binaries like `firebase` from the command line, buy prefixing any invocation with `npx`, so if you wanted to use the firebase command line tools like `firebase login`, you'd run `npx firebase login` instead. This makes sure you're running the version for this specific project which is useful if you have multiple projects.

## Setting up your local config

Once you have the code and dependencies, sure you need to associate it with the project you just created, by running a few commands in the terminal

Note: If you get a `HTTP Error: 404` message when running the below commands, it's likely you're not signed in. You can sign in with `npx firebase login` - you'll be prompted to sign in with a google account.

``` bash
npx firebase login
```

### Initialising your local project

Once you're logged in, you can initialise your local setup:

``` bash
npx firebase init
```

You want:

- hosting (so people can see your site, when you make deploys)
- database (so data it can be saved somewhere, between visits)
- functions (to allow for importing users from airtable or google spreadsheets)

You'll be prompted for some further answers. Add the answers to the corresponding questions below:

#### Database Setup

- database.rules.json - `Y`

#### Functions Setup

- Typescript/Javascript ? `Javascript`
- Use ES Lint to catch bugs? `Y`
- Functions overwrite? `N`
- Install dependencies now? `Y`

#### Hosting Setup

You'll be asked what you want as your public directory. We're using a webpack, a build tool to optimize the assets we serve, and we currently put them into the `dist`, instead of `public`.

- What do you want to use as your public directory? `dist`
- Configure as a single page app? `Y`

### Setting credentials

You'll next need the credentials for either running a local version for development, for carrying out deploys to firebase.

#### Fetching the credentials from firebase:

Go to [project overview > project settings](https://console.firebase.google.com/project/YOUR_PROJECT_NAME/settings/general/)

click "Add firebase to your web app"

Copy the credentials to your clipboard and copy the two sample `env.sample.sh`, in the `envs` directory, like so

```
cp .envs/env.sample.sh .envs/env.myproject.sh
```

Now, add the corresponding credentials to this newly created `env.myproject.sh`  shell script , so for `FIREBASE_DATABASEURL` you'd add whatever was matched by the `databaseURL` in the project settings, and so on.

For airtable, you'll need data the name of the airtable base, and your API key.

Visit https://airtable.com/api to see a list of the airtables you can programatically access - you can see the base key in the url when you visit any of the options, and it will start with `app` followed by an alphanumeric string.

You'll also need your API key from your account page, at:

https://airtable.com/account

```
export AIRTABLE_BASE_DEV="appzxxxxxxxxxxxxx"
export AIRTABLE_API_KEY_DEV="keyxxxxxxxxxxxxxx"
```

#### Fetching the service account for firebase

You'll likely want to be able to run admin commands, like importing the initial set of users, and so on. You can do this by downloading a service account key, from the admin section.

Go to [project overview > project settings > service accounts](https://console.firebase.google.com/project/YOUR_PROJECT_NAME/settings/serviceaccounts/adminsdk).

You can also reach this at the link below:

On the service accounts page, hit GENERATE NEW PRIVATE KEY and save the file in your project, in the `.envs` directory, with a name like `project-name-service-account.json`, so that if your project name was `trying-this-out`, your key would be called `trying-this-out-service-account.json`.

You'll need to add this to your env files, so adding a line like this below for `.envs/dev.sh`:

``` bash
export FIREBASE_SERVICE_ACCOUNT_PATH='trying-this-out-service-account.json'
```


## Running it to test it out

Before you can see anything you need to import users into firebase. The fastest way to do this is to run one of the admin scripts for importing users on your own machine.

Later on, you can trigger this import from a cloud function instead, if you want to regularly import users from an airtable.


```
source .envs/env.myproject.sh
node server/scripts/importUsersAndTags.js
```

Depending how many users you have to import this will take a few minutes to run, as it loops through the users in airtable, and create firebase accounts for them, and builds a data structure for the app to search through.

### Running locally

Once you have users imported, you can try running the constellate application locally - this will connect to the database hosted with google, that you just imported users to. Make sure you're in `cl8-web` then run:

```
npm run serve
```

This will spin up a development server, typically running locally on port 8080. you can visit it by visiting the address below in your browser:

http://localhost:8080


## Deploying somewhere others can see it

Once you see it working, likely you'll want others to be able to see it too.

You can deploy the app by calling the npm command below when in the `cl8-web` directory, but first, you'll need to make sure you're targeting the correct environment to deploy to.

If you have more than one target (maybe you have a dev, or a production environment,or you've set up multiple constellations, you might want to switch betweem them.

You'd set the target by calling:

```
source .envs/env.myproject.sh
```

Where `myproject` is the project you want to deploy the code to,

If you have a target set, you can make your first deploy:


```
npm run release
```

This will make a smaller, compiled version of the javascript for production, then create a new release, which will be visible at:

[http://YOUR_PROJECT_NAME.firebaseapp.com](http://YOUR_PROJECT_NAME.firebaseapp.com)

#### Seeing what the current project and variables are:

If you're not sure what the project is you can check what the current project with firebase is by calling `firebase use` by itself


```
npx firebase use
```

Similarly, you can check what environment variables are set by calling `env` on the terminal, usually filtering it through the `grep` command for say, `FIREBASE`:

```
env | grep FIREBASE
```

## Changing the domain

Once this is working, if you want a different domain name other than `YOUR_PROJECT_NAME.firebaseapp.com` you will need to add an domain entry on firebase then set A-records to point the domain at firebase's servers.

You can do this in the hosting section -> Connect Domain:

https://console.firebase.google.com/project/YOUR_PROJECT_NAME/hosting/main

You can also set the domain email comes from too, in the email authentication section:

https://console.firebase.google.com/project/YOUR_PROJECT_NAME/authentication/emails

In both cases follow the steps in the UI - you will need access to DNS records for your domain name here.
