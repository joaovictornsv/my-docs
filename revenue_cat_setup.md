# Revenue Cat Setup

The RevenueCat docs are very good and detailed. So I will not repeat the same information here.
I will just provide an order of the steps to follow and redirect you to the RevenueCat docs.

## Create a Revenue Cat account

- Go to [Revenue Cat](https://www.revenuecat.com/) and create an account. ([Reference](https://www.revenuecat.com/docs/getting-started/quickstart#1-create-a-revenuecat-account))

## Create a new project in Revenue Cat

- Go to the dashboard and add a new project ([Reference](https://www.revenuecat.com/docs/getting-started/quickstart#%EF%B8%8F-create-a-project)).

## Configure the project in Revenue Cat

In the project page, click on `ADD AN APP` and select the platform.
Below are the configurations for each platform.

### Google Play
1. Provide the name and the package name of the app.
2. Provide the service account file. ([How to create a service account](https://www.revenuecat.com/docs/service-credentials/creating-play-service-credentials#2-create-a-service-account))
3. Setup Google Developer notifications ([Reference](https://www.revenuecat.com/docs/platform-resources/server-notifications/google-server-notifications))
4. Copy and save the public key from RevenueCat project page. It will be the value of the `googlePublicKey` in the `settings.json` file.

#### Products
1. Set up the products in the Google Play Console ([Reference](https://www.revenuecat.com/docs/getting-started/entitlements/android-products)).
2. Copy the product identifiers (e.g. `com.example.monthly:p1m`) and save them. They will be the values of the `googleMonthlyProductId` and `googleAnnualProductId` key in the `settings.json` file.

### Apple
1. Provide the name and the package name of the app.
2. Provide the App-Specific Shared Secret ([Reference](https://www.revenuecat.com/docs/service-credentials/itunesconnect-app-specific-shared-secret))
3. Provide the In-App Purchase Key ([Reference](https://www.revenuecat.com/docs/service-credentials/itunesconnect-app-specific-shared-secret/in-app-purchase-key-configuration))
4. Enable Apple Server to Server ([Reference](https://www.revenuecat.com/docs/platform-resources/server-notifications/apple-server-notifications))
5. Copy and save the public key from RevenueCat project page. It will be the value of the `applePublicKey` in the `settings.json` file.

#### Products
1. Set up the products in the App Store Connect ([Reference](https://www.revenuecat.com/docs/getting-started/entitlements/ios-products))
2. Copy the product identifiers (e.g. `com.example.monthly`) and save them. They will be the values of the `appleMonthlyProductId` and `appleAnnualProductId` key in the `settings.json` file.

### Stripe
1. Create a new account in Stripe ([Reference](https://www.revenuecat.com/docs/getting-started/quickstart#%EF%B8%8F-create-a-stripe-account))
2. Connect RevenueCat with Stripe ([Reference](https://www.revenuecat.com/docs/web/connect-stripe-account))
3. Go to Revenue Cat dashboard and provide the name and select the Stripe account.
4. Configure the webhook inside RevenueCat to listen Stripe events ([Reference](https://www.revenuecat.com/docs/platform-resources/server-notifications/stripe-server-notifications))
5. [Configure the webhook for our app on Stripe](https://dashboard.stripe.com/webhooks). The URL should be `https://<app-url>/stripe-webhook`. We only need to listen to the `checkout.session.completed` event.
   - Copy the webhook secret and save it. It will be the value of the `endpointSecret` in the `settings.json` file.
6. Copy the Stripe secret key (available in the [dashboard](https://dashboard.stripe.com/dashboard)) and save it. It will be the value of the `secretKey` in the `settings.json` file.
7. Copy and save the public key from RevenueCat project page. It will be the value of the `stripePublicKey` in the `settings.json` file.

#### Products
1. Set up the products in the Stripe Dashboard ([Reference](https://www.revenuecat.com/docs/getting-started/entitlements/stripe-products))
2. Copy the product ids (e.g. `prod_xxxxxx`) and save them. They will be the values of the `stripeMonthlyProductId` and `stripeAnnualProductId` key in the `settings.json` file.
3. Also copy the price ids (e.g `price_xxxxxx`) and save them. They will be the values of the `stripeMonthlyPriceId` and `stripeAnnualPriceId` in the `settings.json` file.

### Configure Offerings
Import and organize your products in RevenueCat ([Reference](https://www.revenuecat.com/docs/getting-started/entitlements))

## Webhook
1. Go to `Integrations` section in the sidebar of the project page.
2. Click to add a webhook
3. Provide the URL of the webhook. It should be `https://<app-url>/revenue-cat-webhook`.
4. Provide an Authorization header value. It can be any value. It will be the value of the `authorization` key in the `settings.json` file.
5. Configure to send events only for Production environment.
6. Keep the Events Filter as `All apps` and `All Events`.

You can follow the steps above to create a Dev webhook to test the Sandbox environment.

## Meteor settings example
If you follow correctly the steps above, you may have all the secrets saved.
So, your Meteor settings should be like this:

```json
{
  "public": {
    "revenueCat": {
      "googlePublicKey": "xxx",
      "googleMonthlyProductId": "xxx",
      "googleAnnualProductId": "xxx",

      "applePublicKey": "xxx",
      "appleMonthlyProductId": "xxx",
      "appleAnnualProductId": "xxx",

      "stripePublicKey": "xxx",
      "stripeMonthlyProductId": "xxx",
      "stripeAnnualProductId": "xxx"
    }
  },
  "revenueCat": {
    "authorization": "xxx"
  },
  "stripe": {
    "stripeMonthlyPriceId": "xxx",
    "stripeAnnualPriceId": "xxx",
    "secretKey": "xxx",
    "endpointSecret": "xxx"
  }
}

```
