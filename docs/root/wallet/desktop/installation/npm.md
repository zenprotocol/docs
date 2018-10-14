# NPM Package

Preparing your machine to install the Zen Protocol desktop wallet and the Zen Full Node via npm packages

--------------------------------------------------------------------------------

Please note that the usage of the software is only permitted to anyone who purchased a license during the license sale period. [Sale Terms​](https://www.zenprotocol.com/legal/zen_protocol_token_sale_agreement.pdf)

## Learn how to install and use the wallet with video tutorials:
Video Tutorials


## OSX

1. Install [mono-devel](http://www.mono-project.com/download). If you choose to install via a package manager, add Mono's own repository first.
2. Install brew (<https://www.howtogeek.com/211541/homebrew-for-os-x-easily-installs-desktop-apps-and-terminal-utilities>)
    1. ```xcode-select --install```
    2. ```ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"```
    3. Run this to check if brew is installed properly: ```brew doctor```
3. Open the **Terminal** (can be found from **search** bar)
4. Install lmdb. Enter the command (in terminal) ```brew install lmdb```.
5. [​Install Nodejs](https://nodejs.org/en/download/) (Version >= 6)
    1. Recommended to install using [NVM​](https://github.com/creationix/nvm#installation)
    2. Recommended to install Node LTS (8.9.4) ```nvm install 8.9.4```


## Windows

1. Install [.NET Framework 4.7](https://www.microsoft.com/en-us/download/details.aspx?id=55167).
2. [​Install Nodejs](https://nodejs.org/en/download/) (LTS version recommended)
3. Open the [Command Prompt​](https://www.lifewire.com/how-to-open-command-prompt-2618089)


## Linux

1. Install [mono-devel](http://www.mono-project.com/download). If you choose to install via a package manager, add Mono's own repository first.
2. Install lmdb. The package name is liblmdb0 on Ubuntu and lmdb on Fedora.
    ```sudo apt install liblmdb0```
3. Install Nodejs (Version >= 6)
    1. Recommended to install using [NVM​](https://github.com/creationix/nvm#installation)
    2. Recommended to install Node LTS (8.9.4) ```nvm install 8.9.4```


## Point your npm directory to our repository

Run the following commands in the Terminal / Command Prompt:

```npm config set @zen:registry https://www.myget.org/F/zenprotocol/npm/```


## Installing / Updating

Run the following commands in the Terminal / Command Prompt:

```npm install @zen/zen-wallet -g```


## Running the wallet + full node

Run ```zen-wallet``` from anywhere in your command line (terminal) to start up the wallet

## Running with arguments

### Wipe (Clear Data)

When we launch a new testnet sometimes you will need to wipe your local database in order to sync.

Note that this will NOT delete any data/tokens from the wallet.

You can do this by running: ```zen-wallet wipe```

### Full Wipe  (Clear Data and wallet)

To completely wipe the blockchain + the wallet from your computer run: ```zen-wallet wipefull```
Running without full node

If you would like to run the node independently in a separate tab you can run zen-wallet uionly to launch the wallet without the full node running in the background.

To learn how to run the Zen Node in headless mode click here.
