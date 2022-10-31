## Overview
Orbit is composed of an API and a UI element (released as separate nuget packages, though on the same release schedule). The upgrade procedure essentially involves replacing both elements on the webserver (be that in IIS or an AppService WebApp). There is a wizard for IIS, but it is just as easy to manually update, and is good practice for when you need to do it in Azure (eventually, all systems will be in Azure, so best to learn!). It is also important to remember that there may be SQL scripts to run on the Orbit database as part of the upgrade.

## Prerequisites
- Basic knowledge of how WebApps are deployed in Azure
- Knowledge of using WinSCP to manipulate files deployed in a WebApp
- The FTP deployment credentials for the WepApp that hosts Orbit. 
- Basic proficiency wih VSCode

## 1) Obtain Latest Orbit Release
***Note**: You may need to install and configure nuget. See end of this guide for details*

Use `nuget list faststats` to see the latest version of Orbit.

Run `nuget install FastStatsOrbitAPI` and `nuget install FastStatsOrbitUI`. The packages will be downloaded into  `~/.nuget` 

The folder you need to work with will be named after the version, e.g. `1.10.46`

## 2) Copy System-Specific Config in Existing `appsettings.json` Over to New Version's `appsettings.json`

The most important config file in the API package is the `appsettings.json` file in the root folder of the API:
![[Pasted image 20221028162321.png]]
When upgrading, there will be an existing, working config for the system; you'll need to move these settings into the new appsettings.json file in the new version of the API. The easiest way to do this is by doing a compare in VSCode.

Procedure:
1) Get a copy of the exisitng `appsettings.config`. You can either connect to the WebApp's file system over FTP using WinSCP, or use the AppService Editior in the portal and copy the contents into a new file.
2) Set up a convenient folder structure, and open this in VSCode (ignor `slim` for now, this will be addressed later)
![[Pasted image 20221028164638.png]]
3) Right-click on the copied `appsettings.json` file (i.e. the current working config file), and click 'Select for compare'. Then, right-click the `appsettings.json` file in the new version of the OrbitAPI and click 'Compare with selected'. The exisitng, working config (containing names and connection strings for the relevant system) will be on the left, and the new config will be on the right:
![[Pasted image 20221028173908.png]]

4) Carfully go through the documents, and copy over all system-specific information to the new config:

![[Pasted image 20221028174029.png]]
 Do this with the arrow icon in the middle between the two panes:
 ![[Pasted image 20221028174113.png]]


## 3) Changes to OrbitUI

First, in the OrbitUI folder, open `Orbit\en\assets`

The two URLs should point to the FQDN of the WebApp, and the correct path:

```json
{
  "apiURL": "https://crewfsorbit.planning-inc.co.uk/OrbitAPI/",
  "uiURL": "https://crewfsorbit.planning-inc.co.uk/Orbit/"
}
```

Next, in `Orbit\en\index.html`:

```html
<base href="/Orbit/en/">
```
**Note**: Don't forget the trailing slash

## 4) Updating Databases

SQL scripts are included with some updates, and these need to be run on the Orbit database.

Check for database updates in each new release in `faststatsorbitapi\{version}\UpdateScripts`. If an update has been missed, be sure to run any applicable preceeding updates. 

To check which version of Orbit the system is currently running, go to the Orbit login page:

![[Pasted image 20221028182922.png]]

In this case, moving to version 1.10.46 (the release pictured below) would require `1.10.45.0.sql` and `1.10.45.1.sql` to both be run:
![[Pasted image 20221028183223.png]]

## 5) Deploying the New Version

Before deploying anything, ensure the correspondig folders are renamed to `Orbit` and `OrbitAPI`. 

You'll notice above that there were 'slim'  versions. This simply means that other languages have been removed. This will help speed up deployment.

Because we want to keep the previous working version in case the update is unsuccessful and we need to roll back, we don't want to use deployments from VSCode as these will entirely overwrite to WebApp's directories.*

``` 
* On a side note, Deployment Slots are *currently* more hassle than they're worth as they have different URLs to the production slots, and as we see above, we need to hardcode the URLs in step 3. We may be able to make this more DevOps-friendly if we can dynamically define those settings in the Applications Settings in the portal or by some other means)
```
Instead, use WinSCP to make an FTP connection to the WebApp's file system:

![[Pasted image 20221028183939.png]]

![[Pasted image 20221028184022.png]]

![[Pasted image 20221028184038.png]]

You now have access to the WebApp's file system and can move, edit, rename, etc. as you can in any file explorer. 

Before copying over the new OrbitAPI and Orbit folders, rename the current folders `Orbit_old`/`OrbitAPI_Old`. You can simply roll back to these if anythign goes wrong.

The `web.config` file contains URL rewrite rules and should be left in place.  

You're now ready to drag the new versions of `Orbit` and `OrbitAPI` into `/site/wwwroot`. Once these have uploaded, restart the WebApp, go to the Orbit URL, and pray to the software gods that you can log in.

## Troubleshooting

A rouge setting or a missing trailing slash can be the difference between success and... not successs. Using the browser developer tools will provide useful info if things haven't gone to plan, but below are some symptoms and their causes.

| Issue                                        | Solution                                                |
| -------------------------------------------- | ------------------------------------------------------- |
| Spinning broken image icon on the login page | Check `<base href="Orbit/en/"` in `Orbit\en\index.html` |
|                                              |                                                         |


## Notes
### Install and configure nuget
