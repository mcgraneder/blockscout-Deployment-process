# blockscout-Deployment-process

#1update and upgrade system
````
sudo apt update && sudo apt -y upgrade
````

#install erlang
the steps to install erlang in the blockscout manual deployment guide and the polygon guide wont work as the devs changed the exlixir dependancy to version `1.13`yesterday. so we need to follow another method. we will begin by installing the `asdf` library. make sure we have git first
````
sudo apt install curl git
````
clone the asdf library from github
````
git clone https://github.com/asdf-vm/asdf.git ~/.asdf --branch v0.9.0
````
cd into we now have to edit the `bash.rc` script by pasting the following commands at the end
````
nano ~/.bashrc #open bashrc file
````
paste these at the end
````
. $HOME/.asdf/asdf.sh
. $HOME/.asdf/completions/asdf.bash
````
now confitm `asdf` has been installed correctly by running
````
asdf #you may have to close terminal and reopen
````
next we want to add the erlang plugin so we can install it with `asdf`
````
asdf plugin-add erlang
````
we can now list the versions with 
````
asdf list-all erlang
````
this will list the versions, however if you try to install erlang at this moment it will fail because there are some dependancies we are missing. so we need to install these. they can be installed with
````
sudo apt-get -y install build-essential autoconf m4 libncurses5-dev libwxgtk3.0-gtk3-dev libgl1-mesa-dev libglu1-mesa-dev libpng-dev libssh-dev unixodbc-dev xsltproc fop libxml2-utils libncurses-dev openjdk-11-jdk
````
this should do the trick. you can now install erlang with
````
asdf install erlang 24.3.4.1 #has to be this version blocscout is dependant on it other versions didnt work for me
````
this might take a few minutes. you may see some warnings of packages tha are not found. im pretty sure ive losted all the required ones above but if the installation fails just add the packages that it says in the warning messages. anyway now that we have erlang install make it global
````
asdf global erlang 24.3.4.1
````
now we can instal elixir v1.13. to do so we again need to add the elixir plugin
````
asdf plugin-add elixir
````
now we can install exlixir with
````
asdf install elixir 1.13.4 #this is the version i used but has to be at least 1.13
````
again add to global
````
asdf global elixir 1.13.4
````
and finalyy we have erlang and elixir installed

#add nodeJS repo
now add a nodejs repo with
````
sudo curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -
````

#install rust
install rust with
````
sudo curl https://sh.rustup.rs -sSf | sh -s -- -y
````

#install nodeJS
````
sudo apt -y install nodejs
````

#install cargo
````
sudo apt -y install cargo
````

#install other dependancies
lastly we need to install a few other dependancies
````
sudo apt -y install automake libtool inotify-tools gcc libgmp-dev make g++ git
````

#postgres setup
as i mentioned we cannot connect to the catalog postgres server because the catalog user doesnt have superuser privileges. so i suggested making a test database and using this just to get the setup working and we can worry about the DB later. to setup postgres first get the client so we can work in the shell
````
sudo apt -y postgresql-client
````
now we want to install postgress we do so with
````
sudo apt update #update first
````
````
sudo apt install postgresql postgresql-contrib
````
and we can start the service with 
````
udo systemctl start postgresql.service #this is optional
````
By default, Postgres uses a concept called “roles” to handle authentication and authorization. These are, in some ways, similar to regular Unix-style users and groups.

Upon installation, Postgres is set up to use ident authentication, meaning that it associates Postgres roles with a matching Unix/Linux system account. If a role exists within Postgres, a Unix/Linux username with the same name is able to sign in as that role.

The installation procedure created a user account called postgres that is associated with the default Postgres role. There are a few ways to utilize this account to access Postgres. One way is to switch over to the postgres account on your server by running the following command:
 ````
 sudo -i -u postgres
 ````
 Then you can access the Postgres prompt by running:
 ````
 psql
 ````
 If you are logged in as the postgres account, you can create a new role by running the following command:
 ````
 postgres@server:$ createuser --interactive
 ````
 make sure to make this user a super user. Another assumption that the Postgres authentication system makes by default is that for any role used to log in, that role will have a database with the same name which it can access.

This means that if the user you created in the last section is called sammy, that role will attempt to connect to a database which is also called “sammy” by default. You can create the appropriate database with the createdb command.

If you are logged in as the postgres account, you would type something like the following:
````
postgres@server:$ createdb blockscout
````
To log in with ident based authentication, you’ll need a Linux user with the same name as your Postgres role and database.

If you don’t have a matching Linux user available, you can create one with the adduser command. You will have to do this from your non-root account with sudo privileges (meaning, not logged in as the postgres user):
````
sudo adduser blockscout
````
Once this new account is available, you can either switch over and connect to the database by running the following:
````
sudo -i -u sammy
psql
````
Once logged in, you can get check your current connection information by running:
````
blockscout=# \conninfo
````
this will show you your host connection. should look somthng like
````
utput
You are connected to database "blocscout" as user "blockscout" via socket in "/var/run/postgresql" at port "5432".
````
this is the host we can use to connect to blockscout as out local version. now im not sure if the reason i was using this local version was the reason that the whole process failed it is something to consider. so maybe if you also wanted to try and create your own server in pgAdmin or someething, then you will be able to connect exact;y the way in the polygon guide, because this is what ioriginally did with the catalog db until i realised that i had no superusr privileges.

Part 2 - set environment variables
We need to set the environment variables, before we begin with Blockscout compilation. In this guide we'll set only the basic minimum to get it working. your env should look something like
````
# example:  ETHEREUM_JSONRPC_HTTP_URL=https://rpc.catalog.fi/testnet
export  ETHEREUM_JSONRPC_HTTP_URL=<your polygon-edge json-rpc endpoint>
# example: ETHEREUM_JSONRPC_TRACE_URL=https://rpc.catalog.fi/testnet
export ETHEREUM_JSONRPC_TRACE_URL=<your polygon-edge json-rpc endpoint>
# used for automaticaly restarting the service if it crashes
export HEART_COMMAND="systemctl start explorer.service"
# postgresql connection example:  DATABASE_URL=postgresql://blockscout:Passw0Rd@/var/run/postgresql:5432/blockscout
export DATABASE_URL=postgresql://<db_user>:<db_pass>@<db_host>:<db_port>/<db_name>
# secret key base as per docs https://docs.blockscout.com/for-developers/manual-deployment ( Step 4 )
export SECRET_KEY_BASE=VTIB3uHDNbvrY0+60ZWgUoUBKDn9ppLR8MI4CpRz4/qLyEFs54ktJfaNT6Z221No

# we set these env vars to test the db connection
export PGPASSWORD=Passw0Rd
export PGUSER=blockscout
export PGHOST=/var/run/postgresql
````
Now test your DB connection with provided parameters. Since you've provided PG env vars, you should be able to connect to the database only by running:
````
psql
````

Part 3 - clone and compile Blockscout
now we can finally get to cloning the repo and compiling the nessecary files and doing the setup. i know so much prerequisite stuff before we even start lol but we should be goood to go now. so we start by cloning the blockscout repo
````
cd ~
git clone https://github.com/poanetwork/blockscout.git
````
next cd into the clone and run the following to compile the nessecary mix files with erlang
````
cd blockcout
mix local.hex --force
mix do deps.get, local.rebar --force, deps.compile, compile
````
. note this step make take a few minutes to compile so you will have to wait. not sure exactly what these files are for or what their doing but these are the ones that required erlang and elixir v1.13. next we migrate the database. remeber this will fail if your user doesnt have superuser privileges
````
mix do ecto.create, ecto.migrate
````
this steop will also take a while but shouldnt be as long as the compilation step above. next we want to cd into the app and install node modules
````
cd apps/block_scout_web/assets
sudo npm install
sudo node_modules/webpack/bin/webpack.js --mode production
````
note that the last commadn `sudo node_modules/webpack/bin/webpack.js --mode production` will take a while and it wont spit any output it will just be stalling. this is where i was running into memory issues. now this was most likely because of my virtual box only having 2gb of memory. however if you run into the ame issue you can allocate more heap memory with
````
export NODE_OPTIONS="--max-old-space-size=4096" #this will be enough 4 x 1024 Mbytes
````
when this is complete weve made it by the hard part. but the annoying thing is that all of th enext steps give little to no indication of whether they were successful or not or even what they are doing they all give blank responses. so lets continue just keep this in mind

or this step you need to return to the root of your Blockscout clone folder. build static assets with
````
cd ~/blockscout
sudo mix phx.digest
````
now this command returned an unexpected response from me which im not sure is the correct response or an error, but basically it restured that ethreum_json_rpc was not found along with two other files not found, and the text was in red. hoowever after this the message priceeded to say creating required files so i was not sure what to make of this at the time. just letting you know incase you run into the same scenario. next you need to generate self signed certificates with
````
cd apps/block_scout_web
mix phx.gen.cert blockscout blockscout.local
````

#Part 4 - create and run Blockscout service
In this part we need to setup a system service as we want Blockscout to run in the backround and persist after system reboot. start by creating a service file
````
sudo touch /etc/systemd/system/explorer.service
````
Use your favorite linux text editor to edit this file and configure the service.
````
sudo vi /etc/systemd/system/explorer.service
````
had to do this with vim and aww man i hate vim so different to use lol toook me ages to even figure out how to copy and paste ahahaha. the contents of the file should look like
````
[Unit]
Description=Blockscout Server
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=always
RestartSec=1
User=root
StandardOutput=syslog
StandardError=syslog
WorkingDirectory=/usr/local/blockscout
ExecStart=/usr/bin/mix phx.server
EnvironmentFile=/usr/local/blockscout/env_vars.env

[Install]
WantedBy=multi-user.target
````
next you have to enable starting on system service boot. again these commands are annoying because they return no output so its hard to know if they even work or not
````
sudo systemctl daemon-reload
sudo systemctl enable explorer.service
````
#Move your Blockscout clone folder to system wide location
Blockscout service needs to have access to the folder you've cloned from Blockscout repo and compiled all the assets.
````
sudo mv ~/blockscout /usr/local
````
next create an env file and user the same variables that we defined above when setting up the postgres db connection. now again im not sure if the local db was the issue but this is what i did anyway
````
#just make env version of below expoetrs which i pasted from above
# example:  ETHEREUM_JSONRPC_HTTP_URL=https://rpc.catalog.fi/testnet
export  ETHEREUM_JSONRPC_HTTP_URL=<your polygon-edge json-rpc endpoint>
# example: ETHEREUM_JSONRPC_TRACE_URL=https://rpc.catalog.fi/testnet
export ETHEREUM_JSONRPC_TRACE_URL=<your polygon-edge json-rpc endpoint>
# used for automaticaly restarting the service if it crashes
export HEART_COMMAND="systemctl start explorer.service"
# postgresql connection example:  DATABASE_URL=postgresql://blockscout:Passw0Rd@/var/run/postgresql:5432/blockscout
export DATABASE_URL=postgresql://<db_user>:<db_pass>@<db_host>:<db_port>/<db_name>
# secret key base as per docs https://docs.blockscout.com/for-developers/manual-deployment ( Step 4 )
export SECRET_KEY_BASE=VTIB3uHDNbvrY0+60ZWgUoUBKDn9ppLR8MI4CpRz4/qLyEFs54ktJfaNT6Z221No

# we set these env vars to test the db connection
export PGPASSWORD=Passw0Rd
export PGUSER=blockscout
export PGHOST=/var/run/postgresql
````
if everything went right you should be able to start blockscout with
````
sudo systemctl start explorer.service
````
to check service output
````
sudo journalctl -u explorer.service
````
````
# if netstat is not installed
sudo apt install net-tools
sudo netstat -tulpn
````
now for me the command threw and error with litreally no information on why it failed. it just spat out a lode of lines saying generic error messages and this was really frustrating because it gave me no clue as to where i went wrong. my hunch is that it is the local db connection rather than having a proper server to connect to. i think if you also run into errros the nest way forward would b to create a database and server in pgAdmin. anayways tese are all of the steps i took to do the process and obviosuly there are some issues otherwise i would have gotten connected so i hope its enough to go off and your obviously muchh better than me so hopefully you can take it from here and get over that last hurdle. let me know if you have any questions
