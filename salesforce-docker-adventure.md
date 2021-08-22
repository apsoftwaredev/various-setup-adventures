Here is what I did to set up the sfdx environment. 

You can test it in the salesforce docker (google instructions for installing docker. I also use WLS2, but that isn't necessary)
```
docker pull salesforce/salesforcedx:latest-rc-full
docker run -it salesforce/salesforcedx:latest-rc-full
```

Then started the container and attached to it with vscode using the docker extension

That is all that is needed.  The rest here are just some optional preferences.

then uninstalled the nodejs they have to use nvm (note: you do not have to use nvm, nodejs is already installed, I just wanted to use nvm instead because it is easier to switch node versions)
(Note: run these commands in the docker container terminal in VSCODE, not some other command prompt)
I believe I ran this:

```
rm -rf /usr/local/bin/npm /usr/local/lib/dtrace/node.d ~/.npm ~/.node-gyp /opt/local/bin/node opt/local/include/node /opt/local/lib/node_modules
```
but it might not have worked.
```
rm -rf /usr/local/lib/node*
rm -rf /usr/local/include/node* 
rm -rf /usr/local/bin/node*  
```
Whatever the best way to remove nodejs....
then google the instructions for setting up the nvm current version:
```
HOME=/home/node
USER=node
cd ~
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
HOME=/root/   
nvm --version 
nvm install 14.17.5
npm install --global sfdx-cli@latest-rc  
```

then I added this to my .zshrc (after setting up zsh https://ohmyz.sh/) (note: see salesforce installation instructions for using or switching to the stable npm sfdx version instead of the release candidate rc version)

```
export NVMHOME="/home/node"
export NVM_DIR="/home/node/.nvm"
export NODE_PATH="/home/node/.nvm/versions/node/v14.17.5/bin/node"
export NODE_ENV="Development"
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${NVMHOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvmexport NVM_DIR="/home/node/.nvm"

```
If you try to install the nvm in the root it gives permission errors so you have to make a /home/node to install the global packages



```
 PATH="/home/node/.npm-global/bin:${PATH}" 
NPM_CONFIG_PREFIX="/home/node/.npm-global"
mkdir -p "${NPM_CONFIG_PREFIX}/lib"  
npm --global config set user "${USER}" \  
npm --global --quiet --no-progress install \    
&& npm cache clean --force

   
```

I added this to the path:
```
/home/node/.nvm/versions/node/v14.17.5/bin
```
Then add the vscode salesforce extensions and the SFDX Hardis extension to the container

I guess while I'm at it, I'll leave some instructions on how I uploaded it to docker hub in case someone could use that.  The instructions are a bit opaqueout there but it is basically, 

go on docker hub or whatever place you upload your containers (I'm on docker hub) and create a private repository for your salesforce image (mysalesforce or something)

Once you have your docker container running on your computer (note do not run these following commands in the container. use whatever terminal you used to start the container. I was starting the container on WSL2).
```
docker ps  #to get the container id
docker commit -m "setup and everything working fine" -a "Your Name" <container ID>  <name of your image (same name as the name you chose to give your private repository on dockerhub)>
docker image ls   
docker tag <imageName> <dockerhubusername>/<name of your image>:latest 
docker image ls   # see that all is good
 docker login -u <username>
 docker push <dockeusername>/<imagename>:latest  #note you do not include the <> characters  in commands
 ```

I think that is right. Please feel free to correct me or add to it in the issues and comments, if I'm doing something incorrectly. I'm sure it can be simplified.  Some of the docker commands are redundant I think. Also, you can set env variables when running the docker command. Some of the extensions are a little finicky, but you can submit issues and help those developers. If I have time, I'll make a docker-compose file that sets it all up in one command, unless someone else wants to do that.  That would be nice to have and they aren't very difficult to make.
 
 One other thing I have noticed with this setup that took some time to unravel, to log in to the org I had to use the the `sfdx auth:device:login` and copy and paste the code into the browser.  then  `sfdx force:org:display` to get the session info.  Then copy that info into the vscode command palette ctl+shft+p SFDX authorize Org using session ID". and insert the access token when it asks for the session Id.  

The nice thing about the docker setup is that if you mess something up in your setup or some bad code is executed, it doesn't harm your system, it is is contained in the container.  You can just delete it or roll back the container to an earlier commit and keep on going. It also makes it easier to share development environments and test different system setups with different versions. Plus, you develop in a linux environment, which is better.

Also, I use windows terminal for the terminal in vscode (google it).
