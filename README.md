# How to use gopass with WSL Ubuntu 18.04 and Windows 10 Chrome

1. Install pass, and then install gopass (maybe it's not required to install pass first)
2. Softlink pass to gopass: `sudo ln -s /usr/local/bin/gopass /usr/bin/pass`
3. \[IMPORTANT\]: make sure on an existing user account, to do: `pass recipients add <GPG PUB KEY ID>`
4. Clone your pass-store repo: `gopass clone git@github.com:<user>/<repoName>.git --sync gitcli`

This should be it for getting this to work. Extra steps for getting this to work on WSL with browswerpass. See the following links:
- Browserpass Native: https://github.com/browserpass/browserpass-native
- Browserpass extension https://github.com/browserpass/browserpass-extension (and install from chrome web store)
- pinentry-wsl-ps1: https://github.com/diablodale/pinentry-wsl-ps1
- wsl-autostart: https://github.com/troytse/wsl-autostart

# Installation steps for Browserpass (WSL linux x Windows 10 chrome)
1. install browserpass native, following instructions for wsl
    1. install for WSL first. (download browserpass-linux64-X.X.X.tar.gz, extract and install with:
<pre><code># IMPORTANT: replace XXXX with OS name depending on the archive you downloaded, e.g. "linux64"
make BIN=browserpass-XXXX configure      # Configure the hosts json files
sudo make BIN=browserpass-XXXX install   # Install the app</code></pre>
2. install for Windows, by downloading and running browserpass-windows64-X.X.X.msi.
3. Create `C:\Program Files\Browserpass\browserpass-wsl.bat` with the following contents:
<pre><code>@echo off
bash -c "/usr/bin/browserpass-linux64 2>/dev/null"</code></pre>
4. Edit the hosts json file, and replace `browserpass-windows64.exe` with `browserpass-wsl.bat` that was just created.

# Fixing "Error: Unable to fetch and parse login fields:"
We will make a wrapper script as our pinentry-program which calls `pinentry-curses` when in terminal, and uses `pinentry-wsl-ps1` for anything else (e.g. browserpass from chrome)
1. Follow install steps for pinentry-wsl-ps1:
    1. `cd ; git clone git@github.com:diablodale/pinentry-wsl-ps1.git`
    2. `cd pinentry-wsl-ps1 ; chmod ug=rx pinentry-wsl-ps1.sh`
    3. Create a file ~/.gnupg/my-pinentry.sh with the following code:
<pre><code>
#!/bin/bash
# choose pinentry depending on PINENTRY_USER_DATA

case $PINENTRY_USER_DATA in 
user)
    # echo "using pinentry cli"
    exec /usr/bin/pinentry-curses "$@"
    ;;
*)
    #echo "using pinentry-wsl-ps1 gui"
    exec $HOME/pinentry-wsl-ps1/pinentry-wsl-ps1.sh "$@"
esac
</code></pre> 

4. Edit `~/.gnupg/gpg-agent.conf` and add the following lines of code
<pre><code>
debug 1024
debug-pinentry
log-file $HOME/agent.log

pinentry-program /home/xpirep/.gnupg/my-pinentry.sh
</code></pre>

5. Ensure my-pinentry.sh is executable with `sudo chmod 777 $HOME/.gnupg/my-pinentry.sh`
6. add an alias in your bash profile for pass like this: `alias pass="PINENTRY_USER_DATA=\"user\" gopass"`
7. make sure to restart gpg-agent with these commands: `gpgconf --kill gpg-agent` and `gpg-agent --daemon`
8. I had a bit of trouble, and it might have been caused by using $HOME in scripts, hardcode change it to your actual home.

# starting gpg-agent on startup for browserpass
1. git clone into `C:\wsl-autostart` with `git clone git@github.com:troytse/wsl-autostart.git`
2. follow steps in the repo.
3. I created a file in `/etc/init.d/my_gpg-agent`, which runs `eval $(gpg-agent --daemon)`, and edited `wsl-autostart/commands.txt` to include this file to be called on start up.
4. if you fuck up your `/etc/sudoers` file, its easy to fix for wsl, see https://askubuntu.com/questions/1144326/sudoers-file-syntax-error-on-wsl
