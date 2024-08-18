Service to integrate NUT (Network UPS Tools) with Omada API.

# Description / Architecture
The repository is basically split in three parts. The first one is the UPS connection, it uses NUT to connect to the UPS, the second one is a web API, and the third one uses Omada open API to control Omada devices.

The event handler `ups/nut-event-handler.js` will make an HTTP call to the server, with the UPS status and battery left.

The API server is started by running `index.js`, and will listen to the calls coming from the UPS connection, and will listen to HTTP requests to the `/updateStatus` URL. That URL will do two things, will update the UPS service with the current UPS status, and call a function to execute the desired action.

Omada API which is currently used to switch POE ports on an Omada switch, and called when making the request to the API server.

# Installation and configuration

## Prerequisites
- You can use a small SBC like a Raspberry Pi Zero to run it, or a Docker container. 
- Node 20. This was developed and tested using node 20. It should work with other versions but YMMV. I recommend [nvm](https://github.com/nvm-sh/nvm).
- [pnpm](https://pnpm.io)
- [NUT](https://networkupstools.org)
- Configure your UPS using NUT. This falls outside the scope of these instructions, but you can check NUT docs or GPT it 😉. **Please note the name of the UPS that you set in `ups.conf`, you'll need it later**
- And Omada controller. You can get a OC200 or run software controller on the same SBC.

## Install
- `git clone git@github.com:IAmJulianAcosta/nut-omada.git`
- `cd nut-omada`
- `pnpm install`

## Configure
- Create a new application on your Omada controller, **use authorization Code mode**. [Docs](https://use1-omada-northbound.tplinkcloud.com/doc.html#/home).
- Create a new `.env` file (or make a copy of `.env.example`).
- Configure it with the values from the Omada controller app, and `OMADA_CONTROLLER_URL`.
- Set `NUT_UPS_NAME` with the value from `ups.conf`.
- Set `LOG_FILE` value. Make sure log is writable.
- Set `PORT` value, for the local server. Defaults to 9973.

## Run
- `pnpm run build` to transpile to JS.
- Ensure that the .js scripts are runnable by the users.
- `pnpm start` to start the server.
- Ensure that NUT is configured correctly. You can run `upsc <YOUR_UPS_NAME_IN_ups.conf>@localhost ups.status` to confirm.
- Trigger a change on the UPS status by disconnecting and reconnecting it.

## Users and permissions
The user running the script `ups/nut-event-handler.js` is by default `nut`. Please make sure that the user `nut` and/or group `nut` have read and execute access to scripts, and write access to logs.

# Further development
If you want to customize this and add your own functionality I'd recommend to get a cheap smart plug and connect it to the UPS so you can control its power with an app on your phone.

You also can write your own functions to control your own devices by using the [Omada API]([Docs](https://use1-omada-northbound.tplinkcloud.com/doc.html#/home). 

You are not limited to use this repository for Omada actions, you can use to trigger anything you want. You can add your own actions to `src/ups/actions` and add them to `src/omada/config/actions.ts`.
