# The Twelve Factors
## I. Codebase
> Is the app’s codebase tracked in revision control?

You don’t technically need your source code to be tracked in revision control to use Bluemix. However, a revision control system such as Git will make your life much easier and provide clarity around the codebase to be deployed to Bluemix. This is especially true when working on a team. Be sure to also create a .cfignore file which tells Bluemix which files should not be shipped to Bluemix on deployment. Typically you can just copy-and-paste the contents of your .gitignore file (assuming you’re using Git) into your .cfignore file. At a minimum, you want your node_modules directory and npm-debug.log file (for Node.js apps), as well as any other dependencies that get built with your app, listed in your .cfignore file.

## II. Dependencies
> Are the app’s dependencies declared?

A Node.js app should declare its dependencies in the app’s package.json file. For example:
```
{
  …
  "dependencies": {
    "async": "^1.2.1",
    "body-parser": "^1.11.0",
    "bower": "^1.4.1",
    "cf-deployment-tracker-client": "0.0.7",
    "cfenv": "*",
    "cloudant": "^1.0.0",
    "colors": "*",
    "errorhandler": "~1.3.6",
    "express": "^4.10.6",
    "jsforce": "^1.4.1",
    "jsonfile": "^2.2.1",
    "lodash": "~3.9.3",
    "moment": "^2.10.3",
    "morgan": "^1.6.0",
    "request": "~2.58.0",
    "when": "~3.7.3"
  },
  …  
}
```
For Node.js apps, Bluemix will install npm dependencies on deployment. If one or more dependencies are unchanged from the previous deployment, then Bluemix will restore the node modules from cache. This only works if your node_modules directory is listed in your .cfignore file. If your node_modules directory is not ignored then Bluemix will not build your npm dependencies and instead simply ship the contents of your node_modules directory. Caching of npm dependencies can be disabled in Bluemix by setting the NODE_MODULES_CACHE environment variable to false (this feature was added in a recent update to the Node.js buildpack).

## III. Config
> Are there any configuration settings?

The twelve-factor app recommends storing config settings in environment variables. These config settings can be accessed through the process.env object in Node.js apps. For backing services (see the next section) created from within Bluemix the necessary config settings will be automatically added to a VCAP_SERVICES environment variable. Other environment variables (e.g. for external services) can be added to Bluemix manually.

## IV. Backing Services
> Does the app use any backing services?

The Bluemix catalog includes numerous services (such as IBM Cloudant) on which you can build your app. New services can be created and bound to your app from within the Bluemix dashboard, or by using the Cloud Foundry command line interface. You can also declare which services (by service instance name) you would like bound to your app using the manifest.yml file:
```
---
applications:
- name: twelve-factor-app
  memory: 512M
  instances: 1
  services:
  - twelve-factor-app-cloudant-service
  - twelve-factor-app-dashdb-service
  - twelve-factor-app-dataworks-service
```
  
## V. Build, release, run
> Are there any specific tasks that need to be done to build, release, or run the app?

For the most part, the Node.js buildpack for Bluemix will handle the building (e.g. installing npm dependencies), releasing, and running of your app. Other tasks such as building assets can be added as a postinstall script in your package.json file. An example of installing assets with Bower:
```
{
  …
  "scripts": {
    "postinstall": "./node_modules/bower/bin/bower install"
  },
  …
}
```

## VI. Processes
> Does the app have any processes (e.g. background processes) in addition to the main web process?

Processes in twelve-factor apps should be stateless and share-nothing. Bluemix requires one process named web, which is defined in a Procfile. Bluemix will automatically create a Procfile on deployment if one does not exist. Creating a  Procfile is optional. An example of creating the optional Procfile with the one  web process listed:

```
web: node server.js
```
Bluemix does not currently support background processes–only the one web process. In order to have multiple processes in Bluemix you would need to create a composite application (one codebase which defines multiple Bluemix applications).

## VII. Port binding
> On which port does the app’s web process run?

Bluemix provides a VCAP_APP_PORT environment variable to your app (available as  process.env.VCAP_APP_PORT in Node.js apps). Configure your web server process to listen on this port number. Bluemix will then route HTTP requests from port 80 and from port 443 (for HTTPS requests) to this port.

## VIII. Concurrency
> How much memory does the app need and how many instances are needed in order to achieve the required level of concurrency?

Twelve-factor apps on Bluemix scale horizontally by adding additional process instances to an application. The amount of memory allocated to each process and the number of processes to be created can be defined within the manifest.yml file. For example:
```
---
applications:
- name: twelve-factor-app
  memory: 512M
  instances: 1
```

## IX. Disposability
> Are the app’s processes disposable, meaning can they be started or stopped quickly?

Twelve-factor apps on Bluemix should have disposable processes which can be shut down gracefully with a SIGTERM or shut down suddenly in the case of an underlying system failure. Strive for a crash-only design for your app’s processes. This allows Bluemix to stop or start your processes as needed in order to handle crashes, scaling, or other scenarios.

## X. Dev/prod parity
> What is the app’s development environment and how is dev/prod parity maintained?

You want to take steps to ensure that your local development environment closely matches your production environment on Bluemix. There are several approaches to this including using Vagrant or Docker. At a minimum you want to specify in your  package.json file the Node.js and npm versions to use:
```
{
  …
  "engines": {
    "node": "0.12.x",
    "npm": "2.13.x"
  },
  …
}
```
Tools such as cfenv can help you work with your config settings in both your local development environment and in your production environment.

## XI. Logs
> Are the app’s logs streamed to stdout?

Twelve-factor apps on Bluemix should print their logs to stdout. If you simply use  console.log then this prints to stdout. If you use a logging component then it should be configurable to log to stdout as well. There are several ways to access your Bluemix application logs, including using a third-party log management services.

## XII. Admin processes
> Does the app have any one-off admin processes?

Admin processes can be used for tasks such as running database migrations, running a REPL shell, or running miscellaneous scripts. Bluemix does not currently support ad hoc admin processes. However, you can run an admin process as part of your deployment process. The simplest way to do this in a Node.js app is to add a  install or postinstall script to your package.json file. For example:
```
{
  …
  "scripts": {
    "install": "./admin.js db migrate"
  },
  …
}
```
Note that with this approach your admin process must be idempotent as the process will be run once during each and every deployment

# Source
Bradley Holt
https://developer.ibm.com/clouddataservices/2015/07/17/a-twelve-factor-app-checklist-for-deploying-to-ibm-bluemix/