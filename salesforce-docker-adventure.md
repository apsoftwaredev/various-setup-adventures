Here is what I did to set up the sfdx environment using docker. 

You can test it in the salesforce docker (google instructions for installing docker. https://www.docker.com/  I also use [WLS2](https://docs.microsoft.com/en-us/windows/wsl/install-win10), but that isn't necessary)
```
docker pull salesforce/salesforcedx:latest-rc-full
docker run -it salesforce/salesforcedx:latest-rc-full
```

Then after starting the container I attached to it with vscode using the docker extension. (right click on the running individual container in vscode docker extension, select "Attach Visual Studio Code" 

That is all that is needed, other than adding the salesforce and other extensions for the container as needed.  The rest here are just some optional preferences.

(after setting up zsh https://ohmyz.sh/, again optional) (Note: run these commands following in the docker container terminal in VSCODE, not some other command prompt)
```
apt-get update && apt-get upgrade -y
apt-get install zsh git wget nano
chsh -s /usr/bin/zsh root

```
restart terminal, if you use zsh you will want to change the docker command to use zsh instead of bash, or can type zsh in the bash prompt and it will switch when you startup. or add the zsh command to the ~/.bshrc file, although that makes it more difficult to switch back to bash if you want.

```
$ sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
```

then uninstalled the nodejs they have to use nvm (note: you do not have to use nvm, nodejs is already installed, I just wanted to use nvm instead because it is easier to switch node versions)

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
then google the instructions for setting up the nvm current version (https://github.com/nvm-sh/nvm):
```
HOME=/home/node   # create it if it doesn't eist, mkdir /home/node/
USER=node
cd ~
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash

```

added the following to the /root/.zshrc, you can use nano to edit it, or open the file in vscode

```
export NVMHOME="/home/node"
export NVM_DIR="/home/node/.nvm"
export NODE_PATH="/home/node/.nvm/versions/node/v14.17.5/bin/node"
export NODE_ENV="development"
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${NVMHOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvmexport NVM_DIR="/home/node/.nvm"

```
back to the command line in container after adding that to the /root/.zshrc, or /root/.bshrc if using bash
```
source /home/node/.zshrc
HOME=/root/   
nvm --version 
nvm install 14.17.5
npm install --global sfdx-cli@latest-rc  
```
 (note: see [salesforce installation instructions](https://developer.salesforce.com/docs/atlas.en-us.sfdx_setup.meta/sfdx_setup/sfdx_setup_install_cli.htm#sfdx_setup_install_cli_npm) for using or switching to the stable npm sfdx version instead of the release candidate rc version).  If you try to install the nvm in the root npm gives permission errors, so you have to use /home/node or some other location to install the global packages. npm just doesn't like it when you use the /root folder for home



```
PATH="/home/node/.npm-global/bin:${PATH}" 
NPM_CONFIG_PREFIX="/home/node/.npm-global"
mkdir -p "${NPM_CONFIG_PREFIX}/lib"  
npm --global config set user "${USER}" \  
npm --global --quiet --no-progress install \    
&& npm cache clean --force

   
```

I added this to the path, athough may not be necessary:
```
/home/node/.nvm/versions/node/v14.17.5/bin
```
Then add the vscode salesforce extensions and the SFDX Hardis extension to the container

I guess while I'm at it, I'll leave some instructions on how I uploaded it to docker hub in case someone could use that.  The instructions are a bit opaqueout there but it is basically, 

go on docker hub or whatever place you upload your containers (I'm on docker hub) and create a private repository for your salesforce image (mysalesforce or something)

Once you have your docker container running on your computer (note do not run these following commands in the container. use whatever terminal you used to start the container. I was starting the container on WSL2).
```
docker login -u <username>
docker ps  #to get the container id
docker commit -m "setup and everything working fine" -a "Your Name" <container ID>  <name of your image (same name as the name you chose to give your private repository on dockerhub)>
docker image ls   
docker tag <imageName> <dockerhubusername>/<name of your image>:latest 
docker image ls   # see that all is good
docker push <dockeusername>/<imagename>:latest  #note you do not include the <> characters  in commands
 ```

I think that is right. Please feel free to correct me or add to it in the issues and comments, if I'm doing something incorrectly. I'm sure it can be simplified.  Some of the docker commands are redundant I think. Also, you can set env variables when running the docker command. Some of the extensions are a little finicky, but you can submit issues and help those developers. If I have time, I'll make a docker-compose file that sets it all up in one command, unless someone else wants to do that.  That would be nice to have and they aren't very difficult to make.
 
 One other thing I have noticed with this setup that took some time to unravel, to log in to the org I had to use the the `sfdx auth:device:login` and copy and paste the code into the browser.  then  `sfdx force:org:display` to get the session info.  Then copy that info into the vscode command palette ctl+shft+p SFDX authorize Org using session ID". and insert the access token when it asks for the session Id.  

The nice thing about the docker setup is that if you mess something up in your setup or some bad code is executed, it doesn't harm your system, it is is contained in the container.  You can just delete it or roll back the container to an earlier commit and keep on going. It also makes it easier to share development environments and test different system setups with different versions. Plus, you develop in a linux environment, which is better.

Also, I use [windows terminal](https://www.microsoft.com/en-us/p/windows-terminal/9n0dx20hk701?activetab=pivot:overviewtab) for the terminal in vscode (google using windows terminal in vscode if you would like to try that).

to roll back to an earlier docker commit
```
docker ps -a
docker history <container name>
docker tag <commitimageIdtoroll back to> <imagename>:latest
```

recference: https://github.com/salesforcecli/sfdx-cli

some other helpful links for additional setup:

[Connecting to github with ssh](https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh)

[generating a new ssh key and adding it to the ssh agent](https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)

[github vscode SSH setup guide](https://awsm.page/git/use-github-with-ssh-complete-guide-including-vscode-setup/)

[vscode containers advanced](https://code.visualstudio.com/docs/remote/containers-advanced)

ps. don't forget to give stars on repos if you found something helpful there. Also, don't forget to give ratings to your vscode extensions. It encourages more helpful posts and development of the extensions.
