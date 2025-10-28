# Bike BKK project

This is a simple web application to visualize the bike route and bike sharing location in Bangkok and nearby places along each bike routes. My intention is to help people plan their bike trip and see what places and bike sharing location they can find along bike routes.

# Data
The data is from Bangkok Metropolitan Administration (BMA).

|Data|Source|Year|
|----|----|----|
|Bike Route (เส้นทางจักรยาน)|https://data.bangkok.go.th/dataset/bikeway|2023|
|Bike Sharing Location (จุดจอดจักรยานสาธารณะ)|https://data.bangkok.go.th/dataset/bike_pkn|2025|

# Functions

In this web application, users can perform the following actions:
1. View bike routes and bike sharing location in Bangkok
2. View the information of bike routes and bike sharing location
3. Search for nearby places along each bike routes
    
    This seach function only search for the following places within 500 meters from OpenStreetMap (OSM)

    |Key|Value|
    |----|----|
    |amenity|restaurant, cafe, convenience|
    |tourism|museum, gallery, attraction, zoo|
    |leisure|park|


# Demo application

(To be updated)

# How to run the project

**Note:** This section is from official sveltekit project.

Install dependencies with `npm install` (or `pnpm install` or `yarn`), start a development server:

```sh
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open
```

## Building

To create a production version of your app:

```sh
npm run build
```

You can preview the production build with `npm run preview`.

> To deploy your app, you may need to install an [adapter](https://svelte.dev/docs/kit/adapters) for your target environment.