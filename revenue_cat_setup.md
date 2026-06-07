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

## Meteor settings

Add these blocks to your Meteor project's `settings.json` file using the values collected in the steps above. For local testing, use test/sandbox values from your RevenueCat test project.

```json lines
{
  "public": {
    // ...
    "revenueCat": {
      "googlePublicKey": "xxx",
      "googleMonthlyProductId": "com.example.monthly:p1m",
      "googleAnnualProductId": "com.example.annual:p1y",

      "applePublicKey": "xxx",
      "appleMonthlyProductId": "com.example.monthly",
      "appleAnnualProductId": "com.example.annual",

      "stripePublicKey": "xxx",
      "stripeMonthlyProductId": "xxx",
      "stripeAnnualProductId": "xxx"
    }
  },
  //...
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

## How to test locally

### Pre-requisites

Make sure you are a License tester in the Play Store or have a Sandbox Account in the App Store.

- [App store steps to create a sandbox account](https://www.revenuecat.com/docs/test-and-launch/sandbox/apple-app-store#sandbox-considerations)
- [Play store steps to add a license tester](https://www.revenuecat.com/docs/test-and-launch/sandbox/google-play-store)
  - If you are not a license tester, ask your team admin to add you to the list. Make sure to provide the email that you use in the Play Store.

You also need an account in [RevenueCat](https://www.revenuecat.com/). Ask your team admin to add you to the project.

### Configure local webhooks

- For both web and mobile
  - Start a ngrok tunnel with the following command:
    ```bash
    docker run --net=host -it -e NGROK_AUTHTOKEN=xxxxxx ngrok/ngrok:latest http <local-port>
    ```
    The token is your ngrok authtoken. You can get it from [here](https://dashboard.ngrok.com/get-started/setup).
  - Go to RevenueCat dashboard and add the ngrok URL to the webhook URL.
    - On app page, go to `Integrations` tab and click on `Webhooks`.
    - Click on `Dev` webhook and add the ngrok URL.
      Provide the Authorization header with any value, but remember to add it to your `settings.json` file in the `revenueCat` block (outside the `public` block).
    - Remember to put the URL with the `/revenue-cat-webhook` path (e.g. `https://<ngrok-url>/revenue-cat-webhook`).
- For web only
  - Install the `stripe-cli` package and run the following command:
    ```bash
    stripe listen --forward-to localhost:<local-port>/stripe-webhook
    ```
  - Copy the webhook signing secret and add it to your `settings.json` file.
    ```json lines
    {
      "stripe": {
      /// ...
      "endpointSecret": "whsec_3c1xxxxxxxxxxxxxx"
      },
    }
    ```
