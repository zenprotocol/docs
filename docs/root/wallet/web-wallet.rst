=================
Web Wallet (Beta)
=================

The web wallet allows you to access your wallet and make transactions via a web browser.

----

Security Notice
~~~~~~~~~~~~~~~

Web wallets are inherently less secure than other forms of cryptocurrency wallets. You should not use this web wallet to secure or control any tokens or resources that you cannot afford to lose. The major weakness in any web wallet is that its code is delivered in the form of a web page, meaning that if the server is compromised, the attacker can cause it to deliver phishing code to the user, stealing private key data and using it to steal funds. An additional attack vector is the injection of malicious code into a compromised javascript dependency, for example MobX.js. Please use the Desktop Wallet if you want a higher level of security.

----

.. _Desktop Wallet: desktop-wallet.html

Differences between the Web Wallet and the `Desktop Wallet`_:
---------------------------------------------------------------------------------

.. list-table:: Differences between the Web Wallet and the `Desktop Wallet`_
   :header-rows: 1

   * -
     - Web Wallet
     - Desktop Wallet
   * - **Storing seed**
     - Encrypted using a password of your choice and stored in local storage
     - Encrypted using a password of your choice and stored in folder in your computer
   * - **Validation**
     - Uses a "trusted" remote node to sync with the blockchain and validate the consensus rules, and relays signed transactions on its behalf
     - The client runs a full node and validates the consensus rules of the blockchain independently
   * - **Access**
     - Is via a web browser, which can be compromised in multiple ways such as:
        * having a malicious Chrome Extension​
        * If the server that is serving the client gets hacked, the hacker could deliver a version of the website that steals your seed
        * Phishing - someone could create a website with a domain that looks very similar to the real official wallet domain and serve a malicious version of the wallet
     - You download a desktop application to your computer once from a trusted source

----

Steps for using the web wallet as securely as possible:
-------------------------------------------------------

1. Only use a computer that you are sure is not infected with a virus. Viruses such as keystroke loggers can steal your 24 word mnemonic phrase while you are typing it.
2. You should use a web browser with as little as possible extensions or none at all - and only if you completely trust those extensions. Preferably you should use a browser without any extensions.
3. Bookmark that url of the web wallet https://wallet.zp.io and always access it by clicking the bookmark.

----

Accessing the Web Wallet - https://wallet.zp.io​

You should bookmark the wallet url asap (https://wallet.zp.io) and only access it by clicking the bookmark.

You may run the web wallet client locally independently in order to remove the risk of a phishing attack - here are the instructions for doing so.

Code References
~~~~~~~~~~~~~~~

If you would like to go over the code that secures your 24 word mnemonic phrase and signs transactions you can go over the code references here.
