# Headless site setup

### Overview

This document should be used in tandem with the [Platform Setup Process document](https://docs.google.com/document/d/1S3KytVgeI61se7mGyqPj6M6ABMwPCQuNYpCUqUXZkMA/edit#heading=h.ea7q1ul3ketq), where it lays out setting up accounts for new headless site projects.

### Basic start

* Download (not clone or fork) code template from [https://gitlab.com/dotdevv/templates/headless-framework](https://gitlab.com/dotdevv/templates/headless-framework)
* The `firebase.json` file would typically be left as is and any project-specific targets/project names specified in `.firebaserc`. More on this below.

### Firebase setup

* Create 2 new Firebase projects as according to [Platform Setup Process document](https://docs.google.com/document/d/1S3KytVgeI61se7mGyqPj6M6ABMwPCQuNYpCUqUXZkMA/edit#heading=h.ea7q1ul3ketq)
  * `<Client> Website` for production
  * `<Client> Website Stage` for staging
* Configure project alias to the client's project
  * List Firebase aliases: `firebase use`
  * Clear Firebase aliases: `firebase use --unalias <alias name>`
  * Add the right project aliases: `firebase use --add` and follow prompts
* Configure hosting targets to the client's project
  * Remove existing targets: `firebase target:clear hosting`
  * Add targets for the app, admin and api:
    * `firebase target:apply hosting headless-framework <app/website hosting name>`
    * `firebase target:apply hosting admin-headless-framework <admin hosting name>`
    * `firebase target:apply hosting api-headless-framework <api hosting name>`
  * Alternatively you can also just modify `.firebaserc` file
* Make sure Firebase is using the right project: `firebase use <project ID or alias name>`
* Set up `config.default.json`
  * Take a sample from other existing headless project
  * Change the details:
    * Firebase private key details
    * Sanity project details
    * Shopify store and private app details
* Run `npm run env:set`

### Sanity Studio setup

* Remove project ID from `admin/sanity.json > api.projectId`
* Login to Sanity CLI using client's Sanity account (if not existent yet, then create a Sanity account in [manage.sanity.io](https://manage.sanity.io))
* Run `sanity init` then select `Create a new project` (use the default setting for Production dataset, i.e. make it public)
* Open `https://manage.sanity.io` and login as DotDev Support
* Open the newly created project
* Create a new organisation for the client and edit the project to be under the new client organisation
* Add Custom Studio URL in Settings, set to the default `.web.app` domain from Firebase for the admin hosting (production)
* Add CORS Origin
  * Add the default `.web.app` domain from Firebase for the admin hosting, allowing credentials
  * Both production and staging domains should be added
* Create API tokens (save in 1password)
  * Name `Reactify Sanity` and `Read+Write` permission - for `/admin`
  * Name `Gatsby` and `Read` permission - for `/app`
  * Name `Gitlab CI` and `Deploy` permission - for GitLab CI variable
* Set up an env file for running Sanity studio locally: `/admin/.env.development`
  * After setting up the env file, you should be able to run Sanity Studio locally by running `npm run start` inside `/admin`
  * Note: the local Sanity Studio site might not work on incognito
* Once everything is good, do the first Sanity GraphQL API deployment as this will be required to run Gatsby site even locally
* For completeness' sake, you might also want to deploy Sanity Studio
  * To deploy the site properly, the `.env.production` file needs to be set up
* After first deployment, some of the entries in the dataset might need to be pre-populated because the app relies on some fields not being empty to render properly.
  * Some essential settings:
    * Settings > Organisation > Title & Description
    * Settings > Maintenance > Enabled & Password
    * Settings > Menus > Account & Header & Footer
  * Alternatively, you can fix the broken code to gracefully handles the case when the settings are not yet set

### Gatsby site setup

* Set up an env file for running Sanity studio locally: `/app/.env.development`
* Configure `app/config.default.js`
  * Set the `stores` field with client's Shopify store
  * The field `stores[<shop name>].accessToken` is the private app's storefront token
  * The field `services.firebase` should be taken from the web app config in Firebase as is, except region, which should be `us-central1`
* Once set, local build can be run by `npm run start` inside `/app`
* Create Shopify private app for storefront app
  * App name: `Storefront`
  * Developer email: `support+<clientname>@dotdev.com.au`
  * Enable storefront API and grant all permissions
  * Grant Read + Write access on the following resources:
    * Customers
    * Products
  * Grant Read access on the following resources:
    * Inventory
    * Locations
    * Online Store pages
    * Product listings
    * Resource feedback
    * Store content
* Enable multipass in Shopify store
  * Go to settings > Checkout > Customer accounts

### General

* Go through Gitlab CI config and make sure the deploy commands are referring to the right Firebase projects/aliases
* Try running everything locally, and for Gatsby site, try building locally. There may be some issues with the project template and it is easier to debug and resolve before testing through the CI pipeline
