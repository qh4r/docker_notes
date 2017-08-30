##Dust helpers not loading

Removed Dust.js functionality but leaving this in for future reference.

Inside the dustjs-helpers folder in your node_modules, you will notice dustjs-linkedin is version 2.3.5,
while in the node_modules folder of your app there is another version of dustjs-linkedin which is 2.4.0.
I deleted the older version and then everything works well.

https://github.com/krakenjs/kraken-js/issues/236

##Middleware calls (such as to get productTypes) made multiple times due to favicon

Installed and used server-favicon module to fix this and added to root index.js
Used http://www.favicon.cc/ to generate favicon


##QueryString on URL caused custom middleware calls

The ?discountCode=xxxx param was causing the discount code to load.
It was doing a "return" after res.render() which messed everything up.


##Navbar was collapsing incorrectly when in mobile views.

Was due to the "collapse" class missing from a div <div class="collapse navbar-collapse" id="menu-collapse">

##ng-model properties weren't updating properly with ng-if showing/hiding sections

Used ng-show/ng-hide instead so the objects aren't added/removed to and from the DOM

##Had a problem figuring out doing &&|| conditions in the dust templates. Even considered a custom helper.

If you're spending too much time worrying about logic in templates you're probably doing it wrong. Put that logic
in a controller (or other backend object) and let it send a single property into the template that can be evaluated.

##Had a problem with the Vimeo player giving errors about "videos" being null, cross-domain issues with
the iFrame, etc.

Make sure that the Froogaloop ($f) script isn't creating a player before the iFrame has loaded. Doing that
will cause the cross-domain issue. Do something like this with the player the first time it needs to be created:

if (!player) {
    iFrame.load(function() {
        player = $f(iFrame[0]);
        player.addEvent('ready', videoReady);
    });
}

##express-session isn't for production

Removed Kraken functionality but leaving this in for future reference.

express-session isn't for production so switched to cookie-session. It doesn't currently allow session keys to start
with "_" and Lusca (KrakenJS uses it) sets that type of key for CSRF which breaks things. Here's the temporary fix:

1. Turned off CSRF in node_modules/kraken-js/config/config.json for Lusca
2. Added csurf module into index.js (root) instead
3. Also added res.locals._csrf = req.csrfToken(); for each request into index.js (root) to ensure that value makes it down to the browser with each request as needed.
4. Sessions are now in cookies and CSRF works.

Once Lusca updates and allows the key name to change or cookie-session changes and support "_" at the start of a key I'll update things.

##How can you easily update all npm modules?

$ npm install -g npm-check-updates
$ npm-check-updates -u
$ npm install

##Difference between ~ and ^ in package.json for versions (since it's easy to forget)

https://www.npmjs.org/doc/files/package.json.html

~version "Approximately equivalent to version"
^version "Compatible with version"

##Uncaught TypeError: Cannot read property 'ready' of undefined - froogaloop.js:213

When iframe load() fires, froogaloop tries to return a callback object that doesn't exist.
had to modify the following code (unfortunately):

```
function getCallback(eventName, target_id) {
    if (target_id) {
        //Added this line
        if (!eventCallbacks[target_id]) return null;
        return eventCallbacks[target_id][eventName];
    }
    else {
        return eventCallbacks[eventName];
    }
}
```

##List of MongoDB/Mongoose "recipes"

https://gist.github.com/pasupulaphani/9463004

##How do you handle SIGINT on Windows?

http://stackoverflow.com/questions/10021373/what-is-the-windows-equivalent-of-process-onsigint-in-node-js
http://theholmesoffice.com/mongoose-connection-best-practice/

##Not able to see output of booleans in Handlebars bindings trying to write them out for debugging purposes

Use the Assemble (http://assemble.io/helpers/helpers-logging.html) log helper:

{{log ../../userPurchaseInfo.purchasedAndNotExpired }}

##/etc/hosts file entries not respected by browsers on OSX

Found several potential workarounds here:

http://apple.stackexchange.com/questions/26616/dns-not-resolving-on-mac-os

None seemed to work initially. I went to System Preferences --> Network --> Ethernet --> Advanced and
removed the DNS servers there (they were 208.67.222.222 and 208.67.220.220) and changed it to 127.0.0.1. 
Nothing worked still. Switched back to the original DNS entries and magically the host file entry was picked up in Chrome.
I have no idea if the DNS changes did it or some of the other commands. Will update with more details if I find out anything.

#How do you ssh directly into a specific container?

Run:

docker exec -it <nameOfContainer> sh

#How do you setup authentication with Mongodb?

1. Set DB authSchema to 3 (or higher if required) so tools like RoboMongo will work with authentication:

use admin;
db.system.version.save({ "_id" : "authSchema", "currentVersion" : 3 })

2. Create an admin user in the admin database (http://docs.mongodb.org/manual/reference/built-in-roles/):

db.createUser({user:"dbadmin",pwd:"password", roles: [ {role:"root", db:"admin"} ]})

4. Create a user for the database:

db.createUser({user:"webrole",pwd:"password", roles:[ { role:"readWrite", db:"codeWithDan" } ]})

5. Start Mongo with --auth switch to turn on authentication:

mongod --auth

6. Pass credentials to Mongo using Mongoose in the Node app:

var options = {
    user: username,
    pass: password,
    server: {},
    replset: {}
};

options.server.socketOptions = options.replset.socketOptions = { keepAlive: 1 };

mongoose.connect('mongodb://' + config.host + '/' + config.database, options);

#How to get Docker Compose fired up and running

1. Make sure the VM is running

docker-machine start default
docker-machine env default
eval "$(docker-machine env default)"

2. docker-compose up

#How do you automate running a set of MongoDB commands?

Create a .js file and put the commands in there (see .docker/mongoconfig.js)

Run: mongo mongoconfig.js

#How to cleanly shut-down MongoDB

http://docs.mongodb.org/manual/tutorial/manage-mongodb-processes/

Use shutdownServer()

use admin
db.shutdownServer()

For systems with authorization enabled, users may only issue db.shutdownServer() when authenticated to the admin database or via the localhost interface on systems without authentication enabled.

Use --shutdown

From the Linux command line, shut down the mongod using the --shutdown option in the following command:

mongod --shutdown

#How to backup Mongo?

mongodump --host 127.0.0.1:27017 --db codeWithDan --oplog --out .MongodbBackups/9-14-2015


mongorestore --host 127.0.0.1:27017 --db codeWithDan --oplogReplay .MongodbBackups/9-14-2015/codeWithDan


http://docs.mongodb.org/master/reference/program/mongodump/

#How to create an Azure Linux/Ubuntu Virtual Machine with Docker

1. Login to Azure

    azure login

2. Run the following command to see the available Ubuntu VMs:

    azure vm image list | grep Ubuntu-15_04

3. Create the VM with the Docker extensions (Virtual Machine sizing chart: https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-size-specs)

    azure vm docker create --ssh 22 --vm-size 'Medium' --location 'West US' '<vm_name>' 'b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-15_04-amd64-server-20150910-en-us-30GB' '<vm_username>' '<vm_password>'

4. Run the following command to test the VM image creation and ensure Docker is installed on the VM:

    docker --tls -H tcp://<vm_name>.cloudapp.net:2376 info

If you can't connect then Docker probably didn't make it on the VM (had this happen) so do the following to install it:

1. ssh into your VM:

    ssh <vm_name>.cloudapp.net and login using the VM credentials
    
2. See what version of Linux you have: 
   
    cat /etc/*-release
        
3. Run through the steps here to install Docker: http://docs.docker.com/installation/ubuntulinux
4. Install docker-compose by first doing the following to avoid a permission denied error you might get running curl 
   and then directing the input to /usr/local/bin
   
    sudo chown -R $(whoami) /usr/local/bin
   
5. Now run the commands discussed at https://docs.docker.com/compose/install:
   
    Old way: sudo curl -L https://github.com/docker/compose/releases/download/1.4.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
   
6. Run the following to allow docker-compose to be executed:
   
    chmod +x /usr/local/bin/docker-compose
   
7. Run `docker-compose` and make sure the help docs display
   
#How to install Git on a Linux VM

Run the following:

sudo apt-get install git

#How to ssh into Azure virtual machine

ssh <vm_name>.cloudapp.net and login using the VM credentials


#How to easily install/reinstall Node

https://github.com/brock/node-reinstall

bash dropbox/tools/node-reinstall 5.1.0

#How to remove all stopped containers

docker rm $(docker ps -a -q)

#How to remove all untagged images

docker rmi $(docker images | awk '$2 == "<none>" { print $3 }')

#How to remove all Docker volumes

docker volume rm $(docker volume ls -qf dangling=true)


#Unable to authenticate to MongoDB container

Added the following in the .docker/mongo_scripts/first_run.sh script that the Docker-mongo build runs

echo "Setting authSchema to currentVersion:3"
mongo $ROOT_DB --eval "db.system.version.save({ '_id' : 'authSchema', 'currentVersion' : 3 });"


#Install Redis

brew install redis


#Start Redis with a password

redis-server --requirepass password


#How to Build and Run with Docker-Compose

1. Update config values (localhost --> mongo and localhost --> redis) in config.development.json
2. Set APP_ENV to proper value in Dockerfile-redis (default is "development")

export APP_ENV=development

3. Run docker-compose build
3. Run docker-compose up

#What is the difference between ENTRYPOINT and CMD in DockerFile?

Think of ENTRYPOINT as the binary (an executable file you run for example) and CMD as the commands
that you pass to the executable.

http://crosbymichael.com/dockerfile-best-practices.html






