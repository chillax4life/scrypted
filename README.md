# Scrypted Home Automation

Scrypted is a home automation platform primarily focusing on making camera experiences seamless.
 * Streams load instantly, everywhere: [Demo](https://www.reddit.com/r/homebridge/comments/r34k6b/if_youre_using_homebridge_for_cameras_ditch_it/)
 * [HomeKit Secure Video Support](#homekit-secure-video-setup)
 * Google Home support: "Ok Google, Stream Backyard"

<img width="400" alt="Scrypted_Management_Console" src="https://user-images.githubusercontent.com/73924/131903488-722d87ac-a0b0-40fe-b605-326e6b886e35.png">

## Discord

[Join Scrypted Discord](https://discord.gg/DcFzmBHYGq)

## Supported Platforms

 * Google Home
 * Apple HomeKit
 * Amazon Alexa

Supported accessories: 
 * https://github.com/koush/scrypted/tree/main/plugins

# Installation

There are both [Docker and Local Installation options](https://github.com/koush/scrypted/wiki), as well as guides for connecting to HomeKit and Google Home.

## Personal Access Token

Personal access tokens are used to authenticate API requests and integrate Scrypted with external applications.

### How to Obtain Your Personal Access Token

1. Log into the Scrypted Management Console in your browser
2. Navigate to **Settings** (found in the sidebar under Utilities)
3. Your personal access token will be displayed on the Settings page
4. Click the **Copy** button to copy your token to the clipboard

Alternatively, you can retrieve your token via the API:

```sh
# Replace USERNAME and PASSWORD with your Scrypted credentials
# Replace SCRYPTED_SERVER with your server address (e.g., localhost:10443)
curl -u USERNAME:PASSWORD https://SCRYPTED_SERVER/login
```

The response will include your personal access token:

```json
{
  "username": "admin",
  "token": "your-token-here"
}
```

### Using Your Personal Access Token

Once you have your token, you can use it to authenticate API requests:

```sh
# Use Bearer authentication with your token
curl -H "Authorization: Bearer YOUR_TOKEN" https://SCRYPTED_SERVER/endpoint/@scrypted/core/public/
```

Or use it in Basic Authentication:

```sh
# Use your username and token as the password
curl -u USERNAME:TOKEN https://SCRYPTED_SERVER/endpoint/@scrypted/core/public/
```

**Important:** Keep your personal access token secure. It provides full access to your Scrypted server.

## Development

## Debug the Scrypted Server in VSCode

```sh
# check out the code
git clone https://github.com/koush/scrypted
cd scrypted
# get the dependencies for the server and various plugins
./npm-install.sh
# open server project in VS Code
code server
```

You can now launch Scrypted in VSCode.

## Debug Scrypted Plugins in VSCode

```sh
# this is an example for homekit.
# follow the steps above to set up the checkout.
# open the homekit project in VS Code
code plugins/homekit
```

You can now launch (using the Start Debugging play button) the HomeKit Plugin in VSCode. Please be aware that you do *not* need to restart the Scrypted Server if you make changes to a plugin. Edit the plugin, launch, and the updated plugin will deploy on the running server.

If you do not want to set up VS Code, you can also run build and install the plugin directly from the command line:

```sh
# currently in the plugins/homekit directory.
npm run scrypted-webpack && npm run scrypted-deploy 127.0.0.1
```

## Plugin Development

Want to write your own plugin? Full documentation is available here: https://developer.scrypted.app

