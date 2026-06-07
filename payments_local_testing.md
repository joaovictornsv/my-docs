# Purchases

## How to test locally

### Pre-requisites

Make sure you are a License tester in the Play Store or have a Sandbox Account in the App Store.

- [App store steps to create a sandbox account](https://www.revenuecat.com/docs/test-and-launch/sandbox/apple-app-store#sandbox-considerations)
- [Play store steps to add a license tester](https://www.revenuecat.com/docs/test-and-launch/sandbox/google-play-store)
  - If you are not a license tester, ask your team admin to add you to the list. Make sure to provide the email that you use in the Play Store.

You also need an account in [RevenueCat](https://www.revenuecat.com/). Ask your team admin to add you to the project.

### 1. Add these blocks to your Meteor project's `settings.json` file:

```json lines
{
  "public": {
    // ...
    "revenueCat": { // Make sure to use the publicKey from the test account. These below are from test account
      "googlePublicKey": "",
      "googleMonthlyProductId": "com.example.monthly:p1m",
      "googleAnnualProductId": "com.example.annual:p1y",

      "applePublicKey": "",
      "appleMonthlyProductId": "com.example.monthly",
      "appleAnnualProductId": "com.example.annual",

      "stripePublicKey": "",
      "stripeMonthlyProductId": "",
      "stripeAnnualProductId": ""
    }
  },
  //...
  "revenueCat": {
    "authorization": ""
  },
  "stripe": {  // Make sure to use the price ids and secretKey from test mode. These below are from test mode
    "stripeMonthlyPriceId": "",
    "stripeAnnualPriceId": "",
    "secretKey": "",
    "endpointSecret": "<you will get this in next steps>"
  },
}
``` 
Use the values from your RevenueCat test/sandbox project for the keys above.

### 2. Configure your local webhook

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
