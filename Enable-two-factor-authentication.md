# Background

It is optional, but highly recommended, if you use two or three factor authentication when logging in.

Security in Orko is optionally double-layered.  The first layer is the same username/password/two-factor solution that most cryptocurrency exchanges use.  When you set up an account, you enter a code (or scan an RFID) with a mobile app like Google Authenticator, and when logging in you enter both your username/password and the current code from the app.  In Orko, this two-factor part is optional.  If left blank in configuration, you just leave it blank when logging in.

It's _highly_ recommended that you _do_ use two-factor authentication.  This way, if your password is compromised, a hacker still can't log in.  The guide below talks you through setting this up.

So far, so good.  **However**, there is still a potential weakness.  Being "logged in" to a web application (pretty much all modern ones) relies on the application storing a piece of information in your browser, such as a cookie, which verifies that you have successfully logged in.  Some information from this is sent with each request to the server like a key, and the server makes sure that the key is correct before doing anything.  That's what "logged in" means - your browser has been given a temporary key, much like an electronic hotel room pass, which will work until the server revokes it.  This key is called a JSON Web Token (JWT).

So what happens if someone steals the JWT out of your browser? Yes, they can now pretend to be you and access your account. This potential weakness exists in almost all web applications, including all your favourite cryptocurrency exchanges.

Now, crypto exchanges have the luxury of their code being secret, and presumably (let's hope) can afford to pay security consultants.  They can be pretty sure (again, let's really, really hope) that they have written the browser-side code so that it's very, very hard for some other website to steal your JWT. Orko follows the same best practice, but firstly, as open source code, if there are any weaknesses in security, they are visible for anyone to exploit. Secondly, it's not had the attention of expensive security consultants.

In order to mitigate this, Orko implements **another** (yes, again, optional) layer of security, such that if a hacker steals the JWT, they still can't use it.  When you first connect, you get asked for a 2FA key. If successfully entered, this "whitelists" your IP address.  You can then proceed to normal login.

This means that a hacker needs to have **both** your 2FA key **and** your JWT (or your login details) since the JWT only works from your IP address.

You have the option of setting up:

* **Insecure:** Just username and password
* **Moderately secure:** Username, password and 2FA
* **Recommended security:** Username, password, 2FA and IP whitelisting using the **same** 2FA code 
* **Paranoid:** Username, password, 2FA and IP whitelisting using a **different** 2FA code

Which you use is up to you.

# Instructions

**This urgently needs to be made more accessible to non-technical users. [Can you help](../issues/196)?**

## Install locally first

Currently the utilities for creation of 2FA keys have to be compiled locally:

```
sudo apt-get install maven git
git clone -b release https://github.com/gruelbox/orko.git
cd orko
./build.sh
./start.sh
```

## Create a key

1. Generate a new 2FA secret using `./generate-key.sh`.
1. Note it down - we'll need it when configuring the application.
1. Enter it into Google Authenticator on your phone.

Repeat this if you are intending to use two 2FA codes (see above).

## Local application setup

For manually-configured and local installations, locate this section of your config.yml:

```
#  ipWhitelisting:
#    whitelistExpirySeconds: 86400
#    secretKey: YOURTOKEN
#  jwt:
#    userName: joe
#    password: bloggs
#    secret: CHANGEME!!!!!!!!23423rwefsdf13cr123de1234d1243d1ewdfsdfcsdfsdf12222
#    secondFactorSecret: YOURTOKEN
#    expirationMinutes: 1440
```

Uncomment these lines, replacing the value of:

* `secretKey` with the 2FA code that you want to use for IP whitelisting
* `secondFactorSecret` with the 2FA code that you want to use for login (can be the same as `secretKey`)
* `secret` with a long random string of characters.  Don't copy this from somewhere else - just mash the keyboard randomly.  This is used as a cryptographic seed.
* `userName` and `password` with your chosen username and password. Consider [hashing](Hashing-Passwords) the password just in case your machine is compromised.

## Heroku setup

Set up the following environment variables:

| Variable                           | Set to                                                                                                                                                                                             |
| ---------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `AUTH_TOKEN`                       | The 2FA code that you want to use for IP whitelisting                                                                              |
| `SIMPLE_AUTH_USERNAME`             | The username you want to use when logging in.                                                                                                                                                      |
| `SIMPLE_AUTH_PASSWORD`             | The password you want to use when logging in. Consider [hashing](Hashing-Passwords) the password just in case your machine is compromised.                                                                                                                                                     |
| `SIMPLE_AUTH_SECRET`               | A long random string of characters.  Don't copy this from somewhere else - just mash the keyboard randomly.  This is used as a cryptographic seed.                                                                                                    |                                                               |
| `SIMPLE_AUTH_SECOND_FACTOR`        | The 2FA code that you want to use for login (can be the same as `secretKey`)                                           |