# Welcome to the Skynet Workshop!

Welcome!

In this repo you will find a basic Skynet app online to help you start
developing on Skynet!

The goal of this workshop is to provide developers with examples of the
important concepts of developing an app on Skynet.

_Note: Local Development using MySky can run into issues if your browser security settings are set to "block third-party cookies and data." MySky isn't hosted at localhost, so it is considered "third-party" when your app isn't being run on Skynet. If you see this problem in Chrome or Brave, try visiting [chrome://settings/cookies](chrome://settings/cookies) and adding `localhost` to "Sites that can always use cookies."_

> [Create React App](https://github.com/facebook/create-react-app) is used
> for structuring the project and simplifying deployment, but you don't need
> any knowledge of React to follow the workshop.

## Prerequisites

1. [NodeJS](https://nodejs.org/en/download/) installed.
1. [Yarn](https://yarnpkg.com/getting-started/install) installed. (`npm install -g yarn`)
1. Clone this repo.

## Part 0: Setup

1. Open your terminal to the cloned repo folder and run `yarn install` to
   install the project's dependencies.
1. Run `yarn start` to see our app's starting layout. If your browser doesn't
   launch, visit [localhost:3000](http://localhost:3000). Create React App will
   auto-reload when you save files. (Use <kbd>Ctrl</kbd>+<kbd>C</kbd> in the
   terminal to stop your app.)

## Part 1: Upload a file

We'll first cover the most basic functionality of Skynet, uploading data.

Follow the steps below to update this app to allow the user to upload a file
to Skynet. For this sample app, we'll ask the user to upload a picture.

1.  Install `skynet-js` by running `yarn add skynet-js@beta`
2.  First, you need to import the SDK and initialize a Skynet Client. Open the
    file `src/App.js`, look for where _Step 1.2_ code goes, and paste the
    following code.

```javascript
// Import the SkynetClient and a helper
import { SkynetClient } from 'skynet-js';

// We'll define a portal to allow for developing on localhost.
// When hosted on a skynet portal, SkynetClient doesn't need any arguments.
const portal =
  window.location.hostname === 'localhost' ? 'https://siasky.net' : undefined;

// Initiate the SkynetClient
const client = new SkynetClient(portal);
```

3. Next, create the upload functionality. In the `handleSubmit` function
   (called for when form is submitted), paste the code that will upload a file
   below the _Step 1.3_ mark.

```javascript
// Upload user's file and get backs descriptor for our Skyfile
const { skylink } = await client.uploadFile(file);

// skylinks start with `sia://` and don't specify a portal URL
// we can generate URLs for our current portal though.
const skylinkUrl = await client.getSkylinkUrl(skylink);

console.log('File Uploaded:', skylinkUrl);

// To use this later in our React app, save the URL to the state.
setFileSkylink(skylinkUrl);
```

4. Above this code, uncomment `console.log('Uploading file...');`

5. **Test it out!** If you aren't still running the app, run `yarn start`
   again and try uploading a file. If you open your Developer Console (by
   pressing <kbd>F12</kbd>), the console show helpful messages.

## Part 2: Upload a Web Page

> In Part 1, our app successfully uploaded a file to Skynet, now we'll build
> on that code to upload a web page.

In addition to files, Skynet can receive directory uploads. Once uploaded to
Skynet, any directory with an `index.html` will load in your browser just
like any website. This enables developers to write and deploy their web app,
just by uploading the project's build folder.

1. First, create the upload directory functionality. Back in `handleSubmit`
   inside `src/App.js`, paste this code in the area for _Step 2.1_.

```javascript
// Create the text of an html file what will be uploaded to Skynet
// We'll use the skylink from Part 1 in the file to load our Skynet-hosted image.
const webPage = generateWebPage(name, skylinkUrl);

// Build our directory object, we're just including the file for our webpage.
const webDirectory = {
  'index.html': webPage,
  // 'couldList.jpg': moreFiles,
};

// Upload user's webpage
const { skylink: dirSkylink } = await client.uploadDirectory(
  webDirectory,
  'certificate'
);

// Generate a URL for our current portal
// We'll use a subdomain-style link
const dirSkylinkUrl = await client.getSkylinkUrl(dirSkylink, {
  subdomain: true,
});

console.log('Web Page Uploaded:', dirSkylinkUrl);

// To use this later in our React app, save the URL to the state.
setWebPageSkylink(dirSkylink);
setWebPageSkylinkUrl(dirSkylinkUrl);
```

2. Above this code, uncomment `console.log('Uploading web page...');`

3. **Test it out!** Now the user can submit their name and photo to generate their very own
   web page on Skynet!

## Part 3: Make it Dynamic with MySky

> In parts 1 and 2, you uploaded files onto Skynet. The files at these
> Skylinks are _immutable_, that is, they cannot be modified (or else their URL
> would also change). In this section, we'll use MySky to store editable data
> on Skynet that we can come back to and update.

Right now, if you hover over your image in the certificate, you get a nice
green halo effect. But, we may want to change this later without changing our
skylink. We can do this by saving some editable JSON data to Skynet and
having our web page read the info directly from Skynet.

The first step is hooking up our app to `MySky`, but you'll need a little theory here.

MySky lets users login across the entirety of the Skynet ecosystem. Once logged in, applications can request read and write access for specific "MySky Files" that have a file path. In this workshop, we'll be working with "Discoverable Files" -- these are files that anyone can see if they know your MySky UserID and the pathname for the file.

With MySky, we can write a JSON object to a path, and then tell someone our UserID and the pathname, and they can find the latest version of the data.

Let's setup MySky login and then look at writing data to Mysky files.

1. First we need set the `dataDomain` we will request to from MySky. Replace the line of code in _Step 3.1_ with this.

```javascript
// choose a data domain for saving files in MySky
const dataDomain = 'localhost';
```

2. Now we need to add the code that will initialize MySky when the app loads. This code will also check to see if a user is already logged in with the appropriate permissions. Add the
   following code to `src/App.js` for _Step 3.2_.

```javascript
// define async setup function
async function initMySky() {
  try {
    // load invisible iframe and define app's data domain
    // needed for permissions write
    const mySky = await client.loadMySky(dataDomain);

    // load necessary DACs and permissions
    // await mySky.loadDacs(contentRecord);

    // check if user is already logged in with permissions
    const loggedIn = await mySky.checkLogin();

    // set react state for login status and
    // to access mySky in rest of app
    setMySky(mySky);
    setLoggedIn(loggedIn);
    if (loggedIn) {
      setUserID(await mySky.userID());
    }
  } catch (e) {
    console.error(e);
  }
}

// call async setup function
initMySky();
```

3. Next, we need to handle when a user presses the "Login with MySky" button. This will create a pop-up window that will prompt the user to login. In future versions of MySky, it will also ask for additional permissions to be granted if the app doesn't already have them. Add the following code to `src/App.js` for _Step 3.3_.

```javascript
// Try login again, opening pop-up. Returns true if successful
const status = await mySky.requestLoginAccess();

// set react state
setLoggedIn(status);

if (status) {
  setUserID(await mySky.userID());
}
```

4. We also need to handle when a user logs out. This method will tell MySky to log the user out of MySky and then delete any remaining user data from the application state. Add the
   following code to `src/App.js` for _Step 3.4_.

```javascript
// call logout to globally logout of mysky
await mySky.logout();

//set react state
setLoggedIn(false);
setUserID('');
```

5. Now that we've handled login and logout, we will return to saving data when a user submits the form. Locate and uncomment `console.log('Saving user data to MySky file...');`

6. After uploading the image and webpage, we make a JSON object to save to MySky. Put the following code in the below _Step 3.6_.

```javascript
// create JSON data to write to MySky
const jsonData = {
  name,
  skylinkUrl,
  dirSkylink,
  dirSkylinkUrl,
  color: userColor,
};

// call helper function for MySky Write
await handleMySkyWrite(jsonData);
```

7. We need to define what happens in the helper method above. We'll tell MySky to save our JSON file to our filePath. Put the following code in the below _Step 3.7_.

```javascript
// Use setJSON to save the user's information to MySky file
try {
  console.log('userID', userID);
  console.log('filePath', filePath);
  await mySky.setJSON(filePath, jsonData);
} catch (error) {
  console.log(`error with setJSON: ${error.message}`);
}
```

8. Next, we want the certificate web page to read this data. The code to
   fetch a MySky file is already in the generated page, but it needs to know our userID and the file path in order to find the JSON file. Find the
   code from _Step 2.1_ that says

```javascript
const webPage = generateWebPage(name, skylinkUrl);
```

and replace it with

```javascript
const webPage = generateWebPage(name, skylinkUrl, userID, filePath);
```

9. **Test it out!** Now the user can select the color of the halo which is read from our MySky data! In the next step we'll add loading this data and editing it without re-uploading our image and webpage.

## Part 4: From Shared Data to Shared Logic with DACs

> In part 3, we saw how to save mutable files on Skynet using MySky. In this section, we'll see how to load that data from MySky, and how to use the Content Record DAC to tell other you made new content or interacted with existing content.

DACs provider Javascript libraries that simplify interacting with the web app from your code.

1.  Install `content-record-library` by running `yarn add @skynetlabs/content-record-library`
2.  Next we need to import the DAC. Look for where _Step 4.2_ code goes, and paste the
    following code.

```javascript
import { ContentRecordDAC } from '@skynetlabs/content-record-library';
```

3.  Now, we'll create a `contentRecord` object, used to call methods against the Content Record DAC's API. For _Step 4.3_, paste the following code.

```javascript
const contentRecord = new ContentRecordDAC();
```

4. We need to tell MySky to load our DAC. This also informs it of the permissions our DAC will need for a successful login. Return to the code from _Step 3.2_ and uncomment the following code.

```javascript
await mySky.loadDacs(contentRecord);
```

5. Let's wire up our "Load Data" button, by grabbing data from MySky in the `loadData` method. Add the following code for _Step 4.5_.

```javascript
// Use getJSON to load the user's information from SkyDB
const { data } = await mySky.getJSON(filePath);

// To use this elsewhere in our React app, save the data to the state.
if (data) {
  setName(data.name);
  setFileSkylink(data.skylinkUrl);
  setWebPageSkylink(data.dirSkylink);
  setWebPageSkylinkUrl(data.dirSkylinkUrl);
  setUserColor(data.color);
  console.log('User data loaded from SkyDB!');
} else {
  console.error('There was a problem with getJSON');
}
```

6. Here, we'll add logic for saving edited content to MySky without re-uploading our image and directory. Then, we'll call the Content Record DAC and have it record that we "interacted" with this skylink. This will occur when the "Save Data and Record Update Action" button is pressed, triggering the `handleSaveAndRecord` method. Add the following code at _Step 4.6_.

```javascript
console.log('Saving user data to MySky');

const jsonData = {
  name,
  skylinkUrl: fileSkylink,
  dirSkylink: webPageSkylink,
  dirSkylinkUrl: webPageSkylinkUrl,
  color: userColor,
};

try {
  // write data with MySky
  await mySky.setJSON(filePath, jsonData);

  // Tell contentRecord we updated the color
  await contentRecord.recordInteraction({
    skylink: webPageSkylink,
    metadata: { action: 'updatedColorOf' },
  });
} catch (error) {
  console.log(`error with setJSON: ${error.message}`);
}
```

7. Returning to our logic from before, we want to tell the Content Record DAC that we've created a new certificate when we upload a webpage. Add the following code for _Step 4.7_.

```javascript
try {
  await contentRecord.recordNewContent({
    skylink: jsonData.dirSkylink,
  });
} catch (error) {
  console.log(`error with CR DAC: ${error.message}`);
}
```

8. **Test it out!** Now the user can update the color of the halo which is read from our MySky data! You can also use the [Content Record Viewer](http://skey.hns.siasky.net/) tool to see your content record.

## Part 5: Deploy the Web App on Skynet

Congratulations! You have a fully functioning Skapp! Let's deploy
it and let the world see its wonder! As we mentioned before, deploying an
application is as easy as uploading a directory.

1. For Create React App projects, we need to add `"homepage": ".",` to the `package.json`.

2. Next, we'll return to where we initialized the `SkynetClient` in _Step 1.2_. When deployed to Skynet, we don't want our App to only communicate with siasky.net, instead we want it to communicate with the portal the app is being served from. Find the lines we used to initialize the Skynet Client:

```javascript
const portal =
  window.location.hostname === 'localhost' ? 'https://siasky.net' : undefined;

const client = new SkynetClient(portal);
```

In a production app, you don't need any arguments here (which in JS is the same as passing `undefined` as an argument):

```javascript
const client = new SkynetClient();
```

3. Build the application with `yarn build`

4. Upload the newly created `build` folder to [https://siasky.net](http://siasky.net). (Make sure you select 'Do you want to upload an entire directory?')

5. Now any of your friends can make their own certificates!

## Where to go from here?

Now that you've deployed a Skynet app, there's many things to keep learning!

- You can [learn how to use
  Handshake](https://support.siasky.net/key-concepts/handshake-names) for a
  decentralized human-readable URL like
  [skyfeed.hns.siasky.net](https://skyfeed.hns.siasky.net).

- You can [automate
  deployment](https://blog.sia.tech/automated-deployments-on-skynet-28d2f32f6ca1)
  of your site using a [Github
  Action](https://github.com/kwypchlo/deploy-to-skynet-action).

We're always improving our [Skynet Developer
Resources](https://support.siasky.net/the-technology/developing-on-skynet),
so check that out and join [our Discord](https://discord.gg/sia) to share
ideas with other devs.

## Developing this Workshop

### Available Scripts

In the project directory, you can run:

**yarn start**

Runs the app in the development mode.\
Open [http://localhost:3000](http://localhost:3000) to view it in the browser.

The page will reload if you make edits.\
You will also see any lint errors in the console.

**yarn build**

Builds the app for production to the `build` folder.\
It correctly bundles React in production mode and optimizes the build for the best performance.

The build is minified and the filenames include the hashes.\
Your app is ready to be deployed!

See the section about [deployment](https://facebook.github.io/create-react-app/docs/deployment) for more information.

[![Add to Homescreen](https://img.shields.io/badge/Skynet-Add%20To%20Homescreen-00c65e?style=for-the-badge&labelColor=0d0d0d&logo=data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACAAAAAbCAYAAAAdx42aAAAABGdBTUEAALGPC/xhBQAAACBjSFJNAAB6JgAAgIQAAPoAAACA6AAAdTAAAOpgAAA6mAAAF3CculE8AAAAeGVYSWZNTQAqAAAACAAFARIAAwAAAAEAAQAAARoABQAAAAEAAABKARsABQAAAAEAAABSASgAAwAAAAEAAgAAh2kABAAAAAEAAABaAAAAAAAAAEgAAAABAAAASAAAAAEAAqACAAQAAAABAAAAIKADAAQAAAABAAAAGwAAAADGhQ7VAAAACXBIWXMAAAsTAAALEwEAmpwYAAACZmlUWHRYTUw6Y29tLmFkb2JlLnhtcAAAAAAAPHg6eG1wbWV0YSB4bWxuczp4PSJhZG9iZTpuczptZXRhLyIgeDp4bXB0az0iWE1QIENvcmUgNi4wLjAiPgogICA8cmRmOlJERiB4bWxuczpyZGY9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkvMDIvMjItcmRmLXN5bnRheC1ucyMiPgogICAgICA8cmRmOkRlc2NyaXB0aW9uIHJkZjphYm91dD0iIgogICAgICAgICAgICB4bWxuczp0aWZmPSJodHRwOi8vbnMuYWRvYmUuY29tL3RpZmYvMS4wLyIKICAgICAgICAgICAgeG1sbnM6ZXhpZj0iaHR0cDovL25zLmFkb2JlLmNvbS9leGlmLzEuMC8iPgogICAgICAgICA8dGlmZjpPcmllbnRhdGlvbj4xPC90aWZmOk9yaWVudGF0aW9uPgogICAgICAgICA8dGlmZjpSZXNvbHV0aW9uVW5pdD4yPC90aWZmOlJlc29sdXRpb25Vbml0PgogICAgICAgICA8ZXhpZjpQaXhlbFlEaW1lbnNpb24+NTM8L2V4aWY6UGl4ZWxZRGltZW5zaW9uPgogICAgICAgICA8ZXhpZjpQaXhlbFhEaW1lbnNpb24+NjQ8L2V4aWY6UGl4ZWxYRGltZW5zaW9uPgogICAgICAgICA8ZXhpZjpDb2xvclNwYWNlPjE8L2V4aWY6Q29sb3JTcGFjZT4KICAgICAgPC9yZGY6RGVzY3JpcHRpb24+CiAgIDwvcmRmOlJERj4KPC94OnhtcG1ldGE+Cnr0gvYAAAe5SURBVEgNlVYJbFzVFT3vL/Nt4yXOyiLahF24EMBrszqhNA1EpZWwK0qxZ2xk0apEpaJFNGkzRCC1VYlUJyoisj22EyrFlqBqaGgqiE0QxPaMSyi1E9JS0pRCwGRx7Njz5289702+lWArSZ8zkz/vv3vvufeee+8T+H9WT7WBVb2uEknVXw9XrENEWw6Bm5Hxr4bnz4IuxmHqHwHBu3D81xGYr6Cq5VMlE9ToEN3e+SbF+T8u+hwKD8SuhQjigKhFrp5Pw0CGOv0gAP9xX0CjWksHDA2wvc+51YqM+DWWtJ7E+U7I0xc1Gr4M4hpE3Ed//YPQtW3IMWZjNB1Q2oFpRJBDYz6Nu/zQJqMASD8nM9zgc5ElMOkeg+83oKLjdXQxErXZSFwaQHj4YOPj9GwLJh0a8tPINXMUviA4oEJtiEMQ+klGJwLH/RI0vZJpWAvLmIMztouIbihgtvcQlnT+PoxEFoD0RUDG78IVhivZ0Mhwt1AR403fCiIm0m4/Q76BHu3j3nRZqSn1vavgG+uZgifID4NR8glEYyRWUm6/jIRAqslE2Xa6xRV6K5/DsA/W3U6yDcILDBp0kR8x+LwVd7Wtl8doWmB7k4HiUz5qSgJ0DwnMKxGoHuKbc4RLNi6F8F8iiPmKH0I7gv9+Xob7/zgmsD82DznBQljeMBbvOKsMK82bqEAESEX3wtC/jnHHRlHEgu1uRVl71ngYIXV+hq8gEOiuNZnvDAaidzAFPSRlIQotjcR9ik78MpuCA9GFCLz76OFRLN35pylVAw21mGPtwvGzDolm0ts3UZZYwfcC8bj8yJRceg3VRFBCEIP1teTGLoIgWcW/6PTtmgrhV9uP4vSsFu7eTI8T6G8oU1p97Q2cSD8Pk9S2DJBcP1H7PXH9so1LAWlcRqu0o4uVsluVqCauQ8ZcwfIihDjL7N6tNpZ2biGIVkTwG7z7SAtOjzqoSPwAgbYEZzMbsff6pAKwKu4q4JInX1xyT/Lii2tkXpaoQmxjFYHNiqXrr7zwYE+cnY7KJaD7jz1PDnwHrtfMnP9C6ZOE9dKLyDwHlTs+nLLxdk0uNFZG1Ytnpvakjk0kJEhM2UPClWrKg595B3nGTeTBngsByEPZSpACAQZja5jubnLDIYN/isqOVqWnr24V336FzD6Mqp2vqbPJhuvgubfxnAthfIAl7YfV2fBLpqDgJqEq7q+xbvaRBzDhvjcdQFZAYKB+tepa8vdAbDfm563DyMQ7BLQB5W2vYs9DhZhtNDHY5eyOvTjhdmINq+jtugpKrCPARcg1jpBw+5Be1K8im9UNHKhrRlHOYzjr/Gc6gLDnpxq6oAUlmPDWYlnnMSSjD1O+g4ICo5k/09OnUdXeh75HFsDyfw5NW8Gg7YPjbEEZz8vyzvPr2Kq/hUAUM4ocTu4eHJ14CVfnbsZs6wmMOZ9OJ1HvSBZUxv2Yxm6Fpb2HwWgU5e07kPZvYTfsxdycb7CmDzAyu9iXC3Fn2w8Zzm8yOtfAMI8gFduPPHEnyjqew+LW5UhnHoXGP1NvxQ0FJ6HjUYxleDzInQ4A1dlAaeIjjPNQxs9HXiSBVP19WN55BK98eA9GJjdJirAx1VLZQRr8HTR/DItbamAHlaqBFUX2EuBxDrANnB+HCeRBfPJJEUn9JIF8QA5wWupD0wGMsIXKZRp/Z8uVdhwOGzkB7lb760ikisRmpmA1vTjEPOexT3wfuv4+gTwN3RhGadtKgvwafT6OK/OfQYH1GYF048r5y8grVlXiDtiZSkxMPDADB0gr2Rteq5uDIobfC66iR3LE/hunxhfjnu7RqflxuKEAY8E2vqtTtS3vABmflxH8CuWJbQpwdoRvxtzcG9jOOaKdvzH2L+L0+AtS13QAUiocSslYG1twjKTLzoG0sxHlHc8qAKUcPlPDRhG0me11lmqzBREg7R1C4MfpcZcCkow9TiI+ieKcBeoCM+mO8vzamQGEkzApS0rrYwpkWjSOUpvEqUYp2d/F/j5c4qpmI4H0P7yIfZ6AjWqmxuFtyOQzb0TuW5Ql8PZe9NTkoyB/E9PXhOLcQpxxvj0zAAk5LMdktAV5ZiNO2TYrwmJyPuPbNahoP6giVcNfg8Xa1EgfjP6MZfesVEHjLgksx7jk0h/geRsZkSH2mBL4uAZVHX+5CIBzXHjzu8W4Iqef6m7ktYogdItvTpOUj5GMO5Uh+RXOBdl2+6JVvKw2M9Tl9JadUWi4ghPNkawWz5GE2aEmB/6UgpkeQi6kordRUIaygDm2YQgrG16vl95uh+30Yp99AnFOvea1Fta/arONrybIXRw4c7MXVsjbtIlii/xwS3BXYljOnIsDkKDCATUQLWded9P4AvaHDA0LemUyGlLhKY7rf9AYicXce/5CVs+1NCzUJwg8Es5gY5NV8FuUJn7ElKhquzSA80G81fhltt0EvV/F/Eqms66YYCEiasbzuqfyLfuG4/OLd0BpOJ9VYXsTVPUUw98sVXJJ20R4uSskpTwvL6mB/2M2oFvP3f1p0KM6Bl36pTHn8gIjAaUdXvOCl8mHZ7Bs5/tZrsSl/7KyFAr5/+UtRbRzwnuY63kLZHe8lyAq6PFCNqM5LFabrfZjah7mXg8MYzdKW/+pDMxwh/wf4xZoOPPcKX0AAAAASUVORK5CYII=)](https://homescreen.hns.siasky.net/#/skylink/AQAa1MFnKDhLvVZkwUmOSClsUs-b_7jQbjc9YND8jOosRw)