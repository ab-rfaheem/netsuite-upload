## netsuite-upload VS Code plugin

[![Version](https://vsmarketplacebadge.apphb.com/version/nsupload-org.netsuite-upload.svg)](https://marketplace.visualstudio.com/items?itemName=nsupload-org.netsuite-upload)

**netsuite-upload** is a Visual Studio Code extension that allows you to manage your SuiteScript files directly from VS Code. It also helps you with defining new modules and adding server-side module dependecies.

## Features

### 1. Push and Pull Files and Folders between VS Code and the NetSuite File Cabinet

Right-click a file or folder in the navigation panel to see the context menu options:

- `Pull file from NetSuite` - downloads a file from NetSuite
- `Push file to NetSuite` - uploads a file to NetSuite
- `Delete file in NetSuite` - deletes a file in NetSuite
- `Compare file with NetSuite` - diff your local version with the NetSuite version
- `Pull folder from NetSuite` - Download the folder and all contents from NetSuite
- `Push folder to NetSuite` - Uploads the folder and all contents to NetSuite

![Snippet & commands](img/netsuite_upload.gif)

### 2. NetSuite-Specific Code Snippets & Commands

- `Snippets for module initialization` - type _defineRestlet..._, choose your module type and hit enter
- `Commands for adding new NetSuite/custom dependencies` - open command line (`Ctrl`-`Shift`-`P`) and type
  - _add netsuite dependency_ for choosing of the NetSuite built-in module from the list
  - _add custom dependency_ for defining od custom dependecies

![Snippet & commands](img/snippet_addModule.gif)

### 3. Rebase the Root Folder

You can set the remote destination folder in NetSuite to SuiteScripts (default) or a subfolder of SuiteScripts with the `netSuiteUpload.rootDirectory` setting. This might be useful if you have a folder in the NS file cabinet called `SuiteScripts\Development` where you experiment with scripts before you move them to a `SuiteScripts\Production` folder.

In this kind of work flow, you might do development work within `SuiteScripts\Development`. When you're satisfied, you'd change Settings.json (or Workspace settings in your `.vscode\settings.json` file) to change the base to `SuiteScripts\Production`. Then you'd push up all the files you were working with.

Nothing changes on your local disk due to this setting change. It only modifies the root folder in NetSuite where files get pushed to or pulled from.

This was a requested enhancement. Most people would probably be better off using a sandbox environment as it would be easy to lose track of which files were modified, and which versions of which files were in Development vs Production.

## NetSuite Setup OAuth

### Role Creation / Adding to User

We will create a new role specifically to target the needed permissions for VS Code to work.

To create a new role go under Setup - Users/Roles - Manage Roles - New.

1. Permissions

- Lists
    - Documents and Files - Full
- Setup
    - Allow JS / HTML Uploads - Full
    - SuiteScripts - Full
    - Log in using Access Tokens - Full
    
2. Check **Web Services Only Role** to True

If you wish to upload and download into the **SuiteBundles folder** by changing the `rootDirectory` setting, add the following permission to your Role.

- Setup…SuiteBundler: Full

Save the Role and then add it to the User.

To add a user to the Role, go under - Setup - Users/Roles - Manage Users, find the User you're adding, and go into edit.

In editing mode, go into the sublist and find the sublist tab **Access**. Then go to roles and add the new Role you've just created.

Save and proceed to the next step.

__Keep the name of the Role you created because Script deployment requires it.__

### Script Deployment

Download a copy of the [Restlet](https://github.com/ab-rfaheem/netsuite-upload/blob/master/netSuiteRestlet/vscodeExtensionRestlet.js)

Then upload this to NetSuite.
To upload, go under Scripting - Scripts - New.

Then click the plus sign to add a file from your computer, and choose the RESTlet file you downloaded, Save and Create Script Record.

Please give it a name and save the script record; a button will appear after that. **Deploy Script** Click it and follow the next step.

After you've clicked deploy script, you'll be in script deployment. We will change a few tabs and add a role.

1. Change **Status** to **Released**
2. Go under the sublist tab **Audience** and under Roles, add the Role you just created.

Save and note the URL, and settings.json will use this in **\<RESTlet URL\>**.

Proceed to the next step.

### Access & Integration Tokens

To create an integration token, go under Setup - Integration - Manage Integration - New, give it a name and follow the steps below.

1. Token-based Authentication (Everything else is false)
    - **TOKEN-BASED AUTHENTICATION** - True
2. OAuth 2.0
    - Disable **AUTHORIZATION CODE GRANT** and everything else closes.
3. User Credentials
    - **USER CREDENTIALS** - True

After Saving, this will generate **\<CONSUMER TOKEN\>** and **\<CONSUMER SECRET\>**. Note them into settings.json and go to the next step.

To create an Access token, go under Setup - User/Roles - Access Tokens - New.

- The application name will be The RESTlet Deployment name.
- The User will be the one you gave the Role.
- Role will be the Role created earlier.

Save, and this will generate **\<NETSUITE TOKEN KEY\>** and **\<NETSUITE SECRET\>**. Replace the placeholder in settings.json and go to the next step.

### Netsuite Realm / Account ID

To get the last needed setting for settings.json, go under Setup - Company - Company Information, and in the Main Tab, **ACCOUNT ID** will be replaced with **\<NETSUITE ACCOUNT NUMBER\>** in settings.json.

### Special Notice Regarding the RESTlet

- Future versions of this VS Code Extension may require that you upgrade the RESTlet file in NetSuite. Take note if the extension receives an update, and read the Changelog.
- If you have an old version of the RESTlet in your Production NetSuite instance, and a new version in a SandBox NetSuite instance, be warned. A SandBox refresh might overwrite the SandBox RESTlet with the older version from Production.

## VS Code Setup

### Install the VS Code Extension

The extension can be installed from the VS Code Extensions Marketplace within VS Code.

The Marketplace URL is https://marketplace.visualstudio.com/items?itemName=nsupload-org.netsuite-upload

### VSCode - Open Folder

**Very Important Fact** - Your VS Code project **MUST MUST MUST** be rooted at a folder that maps or corresponds to NetSuite's "SuiteScripts" file cabinet folder. This extension assumes the working root is equivalent to the remote "SuiteScripts" folder.

In VS Code, open the folder that corresponds to your local copy of the **SuiteScripts** folder in VSCode. That folder may not be named "SuiteScripts", but it should be the folder that _corresponds_ to SuiteScripts in the NetSuite file cabiner.

### VS Code Extension Settings

You may manage settings for this extension either in your global user settings for all VS Code projects, or in a project-specific Workspace settings file that can be created or found beneath `.vscode\settings.json`.

Keep reading for required settings relating to authentication and more.

## Authentication Setup

This extension can authenticate to NetSuite and access the RESTlet script using either NLAuth authorization or OAuth 1.0 authorization.

### Authentication Option 1: NLAuth Authorization

Place the following in either Workspace settings or general User settings:

```javascript
{
  // Authentication header
  "netSuiteUpload.authentication": "NLAuth nlauth_account=<ACCOUNTID>, nlauth_email=<LOGIN>, nlauth_signature=<PASSWORD>, nlauth_role=<ROLE>",

}
```

Where:

- ACCOUNTID is your NetSuite account ID number
- LOGIN is your email address used to log into NetSuite
- PASSWORD is your password (make sure you don't commit this file to source control)
- ROLE is the numeric NetSuite RoleID for which you have web service/API permissions. You may need to go look this up in NetSuite Setup…Users/Roles…Manage Roles.

### Authentication Option 2: OAuth

Generating the necessary tokens for OAuth is covered in the NetSuite help site. It's not fun.

- If you wish to use OAuth authentication instead of basic authentication you can leave the authentication header blank and use the OAuth settings properties.
- First, generate an Integration record in NetSuite, make sure the 'token based authentication' scheme is checked, and hang on to the token and secret details.
- Second, log into a role you wish to use for authentication. From the "manage tokens center", generate a new token and secret using the Integration from the previous step.
- Input the 4 values from above (NetSuite key and token and Consumer key and token) in the corresponding settings options.
- Set the `realm` setting equal to your numeric NetSuite account number.

## settings.json

The following demonstrates all possible settings.

```javascript
{
  // Script Deployment URL for the deployed vscodeExtensionRestlet.js
  "netSuiteUpload.restlet": "<RESTlet URL>",

  // Base NetSuite folder path to upload script to (e.g. "SuiteScripts/Developer"). Default if unset is "SuiteScripts".
  "netSuiteUpload.rootDirectory": "<BASE FOLDER PATH>"

  // Temporary folder (e.g. C:\\temp or /tmp) - used for diffing files between local and remote.
  "netSuiteUpload.tempFolder": "<TEMP FOLDER PATH>"

  // AUTHENTICATION - Use either NLAuth or Token Auth.

  // Authentication header for NLAuth. remove or comment out if using OAuth.
  "netSuiteUpload.authentication": "NLAuth nlauth_account=<ACCOUNTID>, nlauth_email=<LOGIN>, nlauth_signature=<PASSWORD>, nlauth_role=<ROLE>",

  // If using OAuth, set all of these.
  // Oauth NetSuite Key or Token ID
  "netSuiteUpload.netSuiteKey": "<NETSUITE TOKEN KEY>",
  // Oauth NetSuite Secret
  "netSuiteUpload.netSuiteSecret": "<NETSUITE SECRET>",
  // Oauth NetSuite Consumer Key
  "netSuiteUpload.consumerToken": "<CONSUMER TOKEN>",
  // Oauth NetSuite Consumer Secret
  "netSuiteUpload.consumerSecret": "<CONSUMER SECRET>",
  // Account number
  "netSuiteUpload.realm": "<NETSUITE ACCOUNT NUMBER>",
}
```

## keybindings.json

You can add keybindings for a number of operations.

By default, two keybindings are pre-set in the Extension: upload and download.

- Upload: Ctrl+n,Ctrl+u
- Download: Ctrl+n,Ctrl+d

You can remap or set new like so in your `keybindings.json` file:

```javascript
{ "key": "ctrl+u",                "command": "netsuite-upload.uploadFile"},
```

## Known Issues and Limitations

The plugin is using a RESTlet for the communication with NetSuite. RESTlets have some governance limitations, meaning NetSuite may throttle API calls if they are sent too rapidly. The current implementation does not deal with this, so there could be problems pulling folders containing a lot of items from NetSuite.
