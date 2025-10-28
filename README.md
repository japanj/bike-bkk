# Bike BKK project

This is a simple web application to visualize the bike route and bike sharing location in Bangkok and nearby places along each bike routes. My intention is to help people plan their bike trip and see what places and bike sharing location they can find along bike routes.

# Project Organization
```
├── LICENSE             <- Open-source license of the project
├── README.md           <- The top-level README for users
├── bike-bkk-web        <- Code for web application development
|   └──  src            <- Project source code
|
├── data                <- All input datasets including raw and processed data
│   
├── pyproject.toml      <- Project metadata (for running Python notebook)
├── uv.lock             <- Lockfile that contains information about Python notebook's dependencies
|
└── data_exploration.ipynb    <- Notebook for exploring raw data and processed raw data
```

# Datasets
The data is from Bangkok Metropolitan Administration (BMA).

|Data|Source|Year|
|----|----|----|
|Bike Route (เส้นทางจักรยาน)|https://data.bangkok.go.th/dataset/bikeway|2023|
|Bike Sharing Location (จุดจอดจักรยานสาธารณะ)|https://data.bangkok.go.th/dataset/bike_pkn|2025|

>Note: License of these data are not specified, but these data are published in BKK Open data portal.

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

<video width="800" controls src="https://github.com/user-attachments/assets/a331859d-c24c-4486-8631-057877e9e7b0" />

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