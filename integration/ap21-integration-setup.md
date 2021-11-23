# AP21 integration setup

## AP21 connector admin app

**Set up Shopify app to be installed in the client's store**

1. Create a new custom app in partner dashboard
   * Note that there will be separate custom apps for staging and production
   * Set the following details on the create custom app form
     * App name: `AP21 (<Client Name> Stage)` or `AP21 (<Client Name>)` for production
     * App URL: `https://app-staging.clientdomain.io/` as will be set up and deployed in the next section
     * Allowed redirection URL(s): `https://app-staging.clientdomain.io/auth/callback`
2. Associate the newly created app with the corresponding Shopify store
3. After the app has been deployed to AWS (see next section), the app can be installed on the Shopify store using the generated link in the app settings page

{% hint style="success" %}
Add DotDev's icon as the app's icon from the `App Setup` section after first creating the app
{% endhint %}

**Deploying admin app to AWS**

1. Create a new secret in AWS secrets manager
   * Typically can reference other projects for what secrets to store
   *   Some typical fields to be included are:

       ```
       {
         "STAGE": "staging", // or "prod"
         "NODE_ENV": "staging", // or "prod"
         "PACKAGE_INDIVIDUALLY": false,
         "AWS_ACCESS_KEY_ID_": "",
         "AWS_SECRET_ACCESS_KEY_": "",
         "AWS_REGION_": "ap-southeast-2",
         "AWS_ACCOUNT_ID": "",
         "AWS_DOMAIN": "clientdomain.io",
         "AWS_CERT": "*.${self:provider.environment.AWS_DOMAIN}",
         "AWS_BUCKET_NAME": "connector-staging.clientdomain.io",
         "AWS_LOG_RETENTION": "1",
         "SHOPIFY_API_KEY": "",
         "SHOPIFY_SECRET": "",
         "SHOPIFY_SHOP_NAME": "clientstore.myshopify.com",
         "APP_NAME": "Apparel21 Connector - Client Name (Stage)",
         "APP_HOST": "app-staging.clientdomain.io",
         "APP_API_DOMAIN": "connector-staging.clientdomain.io"
       }
       ```
   * The name of the secrets are typically `admin-staging` or `admin-prod`
   * The `SHOPIFY_API_KEY` and the `SHOPIFY_SECRET` corresponds to the Shopify custom app created in the previous section
2. Set up certificate for domain of `clientdomain.io`
   * Typically this is created in AWS Certificate Manager on `us-east-1` region (US East N. Virginia)
   *   > To provide a certificate for an edge-optimized custom domain name, you can request [AWS Certificate Manager](https://docs.aws.amazon.com/acm/latest/userguide/) (ACM) to generate a new certificate in ACM or to import into ACM one issued by a third-party certificate authority in the `us-east-1` Region (US East (N. Virginia)).

       [https://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-custom-domains.html](https://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-custom-domains.html)
   * Put the following details to set up the certificate:
     * Domain name: `*.clientdomain.io` (this matches with the value saved in `AWS_CERT` secret value)
     * Validation method: DNS validation
     * Tags: _None_
   * Add the CNAME record on the `clientdomain.io` DNS record, typically managed in AWS Route 53
   * Once set up, it should take a few minutes at most to validate and the certificate issued
3. Set up domain for the new admin app
   * This is only relevant where the admin app domain (e.g. `app-staging.clientdomain.io`) is not yet set up
   * To set up, run `sls create_domain --stage staging`, or `prod` for production deployment
   * This potentially will take some time to be fully ready after first set up
4. Up to this point, the admin app is ready to be deployed on AWS by running `yarn deploy:staging` or `yarn deploy:prod`

{% hint style="warning" %}
Make sure that the `admin/serverless.yml` file is using the client's AWS credentials profile before running any Serverless commands or deploying the app.
{% endhint %}

{% hint style="info" %}
After finished deploying for the first time, it may take a few minutes before the app is actually live, particularly if the app's domain has just got created.
{% endhint %}

## AP21 connector instance

**AWS resources**

1. Set up `[name].io` domain for AP21 connector, setting the subdomain entry to point to the EC2 instance created below. This step typically takes a while and requires some data gathering, hence, best to be done as early as possible.
2. Set up EC2 instance to put the connector app\
   Part of this requires setting up proper VPC security group, allowing inbound HTTP(S), MySQL, Redis and SSH ports access
3. Set up RDS for prod, staging and dev
4. Set up ElasticCache Redis. Typically both prod and staging will use the same Redis instance.
5. Create a new programmatic IAM user, typically with username `ap21-connector` and assign `AdministratorAccess` policy on it. Take note of the access key and secret access key and store them in 1Password.
6. Set up SQS queues for prod, staging and dev, each of which has the following queues: `default`, `api`, `dispatch`, `webhook`

**Non-AWS resources**

1. Create a new server connection in ServerPilot to manage deployment to the EC2 instance created earlier
2. Create separate apps for prod and staging on the server instance, each having their own domain names (which needs to be wired to the EC2 instance using subdomains set up in step 4 above for each app)
3. Configure Gitlab deployment pipeline for both prod and staging

**Initiating fresh installation (AP21 connector v3 - using Composer package)**

1. Follow the guide in this Wiki page: [https://gitlab.com/dotdevv/products/php-apparel21/-/wikis/Installation-guide](https://gitlab.com/dotdevv/products/php-apparel21/-/wikis/Installation-guide)
2. Up until arranging repository, set up Gitlab CI script to automatically deploy to staging and production. Can refer to other projects, such as Licensing Essentials, for a starter on the Gitlab CI setup.
3. Deploy files to staging, before proceeding to setting up `.env` file step in the Wiki
4. Proceed with setting up `.env` file and config files, up until migrating database
5. Run connector install commands:
   1. Set up a blank config files: `php artisan connector:install --config`
      * Copies off config file templates
      * This is essentially the same as calling:
        * `vendor:publish --tag=connector-config`
        * `vendor:publish --tag=connector-custom-config`
   2. Load stores to database: `php artisan connector:install --store`
      * Saves AP21 store locations to database
   3. Setup inventory buffer: `php artisan connector:install --inventory`
      * Initialises inventory buffer and webstore ID in cache
      * It will fail if they are already set up
   4. Setup Shopify webhook: `php artisan connector:install --webhook`
      * Set up Shopify webhooks
      * This operation is idempotent. If webhooks are already set up, it will just print warnings.
      * See the note below for configuring webhooks in `config/connector-custom.php` file.
6. Setup supervisor: refer to the section below
7. Deploy admin app: refer to the section below
8. Create and install admin app for Shopify store

{% hint style="info" %}
Webhooks must be configured in `config/connector-custom.php` to be properly set up by the `connector:install` command. The following is a common webhook configuration:

```
'webhooks' => [
    [
        'topic' => 'orders/paid',
        'address' => env('APP_URL'). '/webhook/order-create',
        'format' => 'json'
    ],
    [
        'topic' => 'customers/create',
        'address' => env('APP_URL'). '/webhook/customer-create',
        'format' => 'json'
    ],
    [
        'topic' => 'fulfillments/create',
        'address' => env('APP_URL'). '/webhook/fulfilment-create',
        'format' => 'json'
    ],
    [
        'topic' => 'customers/update',
        'address' => env('APP_URL'). '/webhook/customer-update',
        'format' => 'json',
        'metafield_namespaces' => [
            'ap21'
        ]
    ],
],
```
{% endhint %}

{% hint style="info" %}
Active online store should be set through the Shopify admin app (check the "Ship from store" box for the relevant store). Manually toggling the database entry via client tool like Tableplus will not trigger the connector to cache the store ID.
{% endhint %}

**Setting up gift card integration**

In order to have the connector work with gift card integrations, a custom Shopify gift card product needs to be created. The gift card product should be set up as follows:

* It should be a normal product rather than Shopify's special gift card product so that a Shopify gift card will not be issued when customer purchased it online.
* The variants should have its `option2` set to `Digital` or `Physical`.

Update the connector configurations with Shopify product ID, AP21 gift card SKU ID and voucher types and run the following command:

```
php artisan connector:install --giftcard
```

**Setting up cache configurations**

Typically there are 2 cache configurations set up in the Laravel project (`default` and `cache`), which can be viewed in the `redis` section of the `config/database.php` file. The connector uses 2 further cache configurations for customers and orders. To set them up, make sure to add the following entries in the `redis` section of the configuration file:

```
'redis' => [
    ...

    'customer' => [
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => env('REDIS_CACHE_DB', 2),
    ],

    'order' => [
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => env('REDIS_CACHE_DB', 3),
    ],
],
```

**Setting up supervisor**

Supervisor will be responsible for managing PHP process that handles events in SQS as dispatched by the connector.

{% hint style="info" %}
Supervisor can be installed in the Ubuntu-based servers by the following command:

```
sudo apt update
sudo apt install supervisor
```
{% endhint %}

If copying off from other projects like Licensing Essentials, there should already be a supervisor configuration file for both staging and production under within `supervisor/` folder in the project root. What's left to be done is to make sure the job definitions in the config is referring to the right directory for the connector and the right queue names as set up previously in AWS.

Once the supervisor configuration files are set, simply update the main supervisor config file, typically located in `/etc/supervisor/supervisord.conf`, and modify the following entry:

```
[include]
files = ... # This should be path to the config files for staging and production, separated by space
            # Alternatively, we can symlink the config files to `/etc/supervisor/conf.d/` instead (typically with .conf extension)
```

Reload supervisor by running `sudo supervisorctl reload`. After the daemon is restarted, verify that the configuration files have been loaded properly by running `sudo supervisorctl status` and you should see the processes running as specified by the loaded config.

**Setting up cron job runner**

Scheduled tasks by default wouldn't run unless the Laravel cron runner is scheduled to run every minute. To set it up, open the cron settings by the command:

```
sudo crontab -e
```

In the editor that opened up, add the following line:

```
# Note that the directory should be adjusted to point to the right connector installation folder
* * * * * cd /srv/users/serverpilot/apps/shopify-ap21 && /usr/bin/php artisan schedule:run >> /dev/null 2>&1
```

**Installing and configuring admin app**

1. Copy admin app files from other v3 project, such as Licensing Essentials. The admin app is typically client-agnostic so it is safe to copy around with minimal changes if at all.
2.  Add secrets to secret manager in AWS for production and staging. The secret name should be in the following format: `admin-[stage]`. Refer to the Serverless deploy command to see the exact name of the stage (the `--stage` option). Typically the stages are `staging` and `prod`. The secret keys to be added are:

    ```
    STAGE
    NODE_ENV
    PACKAGE_INDIVIDUALLY
    AWS_ACCESS_KEY_ID_
    AWS_SECRET_ACCESS_KEY_
    AWS_REGION_
    AWS_ACCOUNT_ID
    AWS_DOMAIN
    AWS_CERT
    AWS_BUCKET_NAME
    AWS_LOG_RETENTION
    SHOPIFY_API_KEY
    SHOPIFY_SECRET
    SHOPIFY_SHOP_NAME
    APP_NAME
    APP_HOST
    APP_API_DOMAIN
    ```

    The secrets stored in the secret manager will be exposed to the admin app as environment variables.
3. Run `yarn install && yarn deploy:[stage]`

**Setup CORS**

The connector, by default, would not allow requests from cross origin. This means the admin app wouldn't be able to access the connector's endpoints because they live on different domain names. To resolve this, install the CORS Laravel middleware on the connector by running this command (and commit the resulting `composer.lock` file).

```
composer require fruitcake/laravel-cors
```

After installing the package, add the middleware to the `app/Http/Kernel.php` on the `$middleware` protected class variable:

```
protected $middleware = [
    ...
    \Fruitcake\Cors\HandleCors::class,
];
```

Add the CORS config by running:

```
php artisan vendor:publish --tag="cors"
```

Finally, add the `*` in the paths option of the generated CORS config file:

```
return [
    ...
    'paths' => ['*'],
    ...
];
```
