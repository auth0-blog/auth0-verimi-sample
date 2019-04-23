# Auth0 Verimi Sample

![Auth0 Verimi Sample](img/fezbot2000-71727-unsplash.jpg)

## Introduction

### Purpose

The purpose of this sample project is to demonstrate how a web developer, using **Auth0**, can add the identity provider **Verimi** as an option for their end users to signup and login to their web application.

### About Verimi

**Verimi is your secure ID provider**

Verimi is the trusted cross-sector european identity platform making processing digital user data easy, while identifing and verifing your users quickly and seamlessly. Verimi helps you meet your sector’s regulatory requirements in accordance with the Money Laundry Act (ZAG license by BaFin), being compliant to KYC standards as well as to European standards (GDPR, elDAS), being ISO/IEC 27001-certified and complying to the requirements of the Federal Network Agency.
Verimi offers the following services: simple registration process, in-line verification, secure login, 2-FA, qualified digital signature and payment.
Verimi is backed by strong partners like Samsung, Deutsche Telekom, Allianz, Axel Springer, Daimler, Deutsche Bank..

* https://verimi.de/en/

### About Auth0

**Auth0 is an Identity as a Platform (IDaaS) service provider.**

Auth0 is Identity made simple and secure. With Auth0, you take perhaps the riskiest endeavor your engineering organization will face, managing and securing the identities of your customers and employees, and abstract away this complexity and risk with our Identity Platform. Our standards-based Identity-as-a-Service platform is  built for developers, by developers so that your engineering organization can focus on building the solutions that delight your customers and drive your revenue. Our customers find that what typically either takes months to deliver or simply cannot be delivered via internal or external solutions, takes them only days to deliver with Auth0 due to our extensive SDKs, intuitive API, extensible architecture and our simple management dashboard. This is why Auth0 is the leader in developer-focused Identity Management, with more than 5,000 customers trusting us with billions of transactions everyday.

* https://auth0.com

## Configuration

### Prerequisites

We assume that you already have registered an Auth0 account and setup a tenant. If not, register a free account and tenant at https://auth0.com
Also, we assume that you already have a client app setup in Auth0. If not, you can refer to the steps and screenshots [further below](#auth0-backend).

Furthermore, we assume that you have already registered a service provider account at Verimi. If not, you can inform yourself about the possibilities here: https://verimi.de/en/partner and contact Verimi by email, referring to Auth0: partner@verimi.com.  

### Configuration on Verimi side

1. Verimi will send you a welcome email requesting contact data (for sending the api access data), your redirect urls (in order to  whitelist them) and you company logo (for co-branded flow)
2. Verimi provides you with an api access to the UAT test environment by email and sms (base URL: https://verimi-uat-web.coretransform.com)
3. You test your integration and do a short acceptance test with Verimis integration team
4. You provide content if you want to be listed as a service provider in the customer login area
5. Verimi will provide you with api access for the live environment (base url: https://web.verimi.de)
6. Verimi will coordinate joint PR and Marketing measures, if you're interested
6. You inform Verimi about the golive date and switch your integration to live environment

This can be done within 1-2 days.

### Configuration on Auth0 side

1. Log into the Auth0 Dashboard.
2. Go to `Extensions` in the left-hand menu.
3. Make sure that the `Custom Social Connections` extension is installed. If not, install it by clicking on it.
![Placeholder](img/custom-social-connection-1.png)
4. Once installed, click on the extension to configure it.
5. The upcoming screen shows an overview of pre-configured but disabled custom connections. Press the orange button labeled `+ New Connection` to create and configure a new connection.
![Placeholder](img/custom-social-connections-2.png)

6. In the upcoming configuration screen, enter the following values:

**Name**: `Verimi`

**Client ID**: `<enter the ClientID you retrieved from Verimi>`

**Client Secret**: `<enter the Client Secret you retrieved from Verimi>`

**Fetch User Profile Script**:

```javascript
function(accessToken, ctx, cb) {
  request.get('https://verimi-uat.coretransform.com/userinfo', {
    headers: {
      'Authorization': 'Bearer ' + accessToken
    }
  }, function(e, r, b) {
    if (e) return cb(e);
    if (r.statusCode !== 200) return cb(new Error('StatusCode: ' + r.statusCode));
    var profile = JSON.parse(b);
    cb(null, {
      user_id: profile.sub,
      given_name: profile.name.split(' ')[0].trim(),
      family_name: profile.name.split(' ')[1].trim(),
      name: profile.name,
      email: profile.email,
      email_verified: profile.email_verified
    });
  });
}
```

**Authorization URL**: `https://verimi-uat.coretransform.com/oauth2/auth`

**Token URL**: `https://verimi-uat.coretransform.com/oauth2/token`

**Scope**: `openid email name`

**Custom Headers**:

Since Verimi requires *Basic Authentication* for authentication, but the *Auth0 Custom Social Login* extension submits the credentials via post body per default, we need to create a custom header for the Authorization and add it to the configuration.

The hash is created by Base64-encoding the `clientID` and `clientSecret` concenated with a ``:``. So the pseudo code to generate this would be:

`base64encode(clientId + ":" + clientSecret)`

so an example could then look like:

```
{
  "Authorization": "Basic NDQyNWM3NGItN2VhOS00MDI0LTk4OGQtY2JmZmRiZjQwNGM1OkhISEh6bHQzZ1JORHV5TGkzbHJmRC1YMWpjTnpfUG5wd1RjYTloc1lySHo4cFlMblVSNVpucmxaMEpKSko="
}
```

In the end, it should look similar to this:

![Placeholder](img/Screen_Shot_2019-04-01_at_1.52.57_PM.png)

You can try out the connection by pressing the `Try` button and, upon success, get a screen like this:

![Placeholder](img/Screen_Shot_2019-04-01_at_2.35.07_PM.png)

Under the `Apps` tab, make sure that your client/application is enabled for this custom social connection, otherwise enable the checkbox.

![Placeholder](img/Screen_Shot_2019-04-01_at_1.53.50_PM.png)

After hitting the `Save` button and returning to the overview of custom social connections, you should see the newly added one listed and active here:

![Placeholder](img/Screen_Shot_2019-04-01_at_1.53.37_PM.png)


7. At last, adjust the `Universal Login Page` configuration and add a few lines of custom css to make the Verimi button appear with the respective logo and coloring.
In the Auth0 dashboard, go to `Universal Login Page` > `Login` (https://manage.auth0.com/#/login_page) and enable the `Customize Login Page` checkbox. Adjust the source code of the login page replacing the `theme` section in the JSON object with the following lines:

```
      theme: {
        logo: config.icon,
        primaryColor: colors.primary ? colors.primary : 'green',
        authButtons: {
          "Verimi": {
            displayName: "Verimi",
            primaryColor: "#0E956E",
            foregroundColor: "#fff",
            icon: "https://raw.githubusercontent.com/auth0-blog/auth0-verimi-sample/master/img/icon.png"
          }
        }
      },
```

Before:

![Placeholder](img/login_before.png)

After:

![Placeholder](img/login_after.png)


## Screenflow

### Screencast

A screencast of the authentication flow is available here: [Screencast](https://youtu.be/qh3L-BQTQBU)

### Authentication

When pressing the login button within the client application,...

![Placeholder](img/Screen_Shot_2019-04-02_at_10.37.07_AM.png)

...Auth0 provides a login/signup widget with options to login/signup via username and password or via social login, such as Verimi.

![Placeholder](img/Screen_Shot_2019-03-25_at_4.40.59_PM.png)

When choosing Verimi (we're assuming the end user already has an account at Verimi), he will be asked to login at Verimi, providing email address...

![Placeholder](img/Screen_Shot_2019-03-25_at_4.41.29_PM.png)

... and password:

![Placeholder](img/Screen_Shot_2019-03-25_at_4.41.41_PM.png)

Upon successful login, he will be asked give consent to Verimi to grant Auth0 access to his personal information:

![Placeholder](img/Screen_Shot_2019-03-25_at_4.41.49_PM.png)

![Placeholder](img/Screen_Shot_2019-03-25_at_4.42.00_PM.png)

Only if the Service Provider requests one of the scopes "idcard", "passport" or "driverlicense" (and by that personal and sensitive data), then Verimi requires the user to confirm the data transfer by 2FA via the Verimi (ios/android) app, which can be downloaded and used for this confirmation without aborting the flow.

![Placeholder](img/Screen_Shot_2019-03-25_at_4.42.10_PM.png)

![Placeholder](img/IMG_0012.PNG)

After confirming, the demo client asks for consent to access the Auth0 tenant:

![Placeholder](img/Screen_Shot_2019-04-01_at_1.28.16_PM.png)

After consent is given, the browser redirects to the application, having received an ID token and information about the user:

![Placeholder](img/Screen_Shot_2019-04-02_at_10.37.00_AM.png)

## Auth0 Backend

### Client App Creation in Auth0

In our example, we chose a `Single Page Application (SPA)` based on `Angular` delivered from a Node.js web server. You can choose any other technology stack as well though.

In our case, the client application is created and configured as follows in the Auth0 backend:

1. Create the application by going to the `Applications` section in the dashboard and click the `+ Create Application` button.

![Placeholder](img/applications-create.png)

2. Give the application any name you like and choose `Single Page Web Application` as the type.

![Placeholder](img/app-quickstart-1.png)

3. Next, choose `Angular` as the technology.

![Placeholder](img/app-quickstart-2.png)

4. Afterwards, you're given the choice to either integrate Auth0 into an existing app, or download a complete sample. Choose the latter and download the `.zip` file.

![Placeholder](img/app-quickstart-3.png)

5. Download and unzip the `.zip` file anywhere on your computer. Then, enter a local terminal window and switch to the folder where you extracted the project into.

To build and run the project, enter:

`npm install`

followed by

`npm start`

This should spin up a local Node.js web server, making the application available at `http://localhost:3000`.

6.

Returning to the Auth0 dashboard, click the `Settings` tab of the app.

In the application settings:

* set `Application Type` to `Single Page Application`.
* set `Allowed Callback URLs` to `http://localhost:3000/callback`
* set `Allowed Logout URLs` to `http://localhost:3000`

![Placeholder](img/app-settings.png)

7.

As for the connections, enable any (other) connections that you want enabled besides `Verimi`. Note that custom social connection don't show in this screen, they are configured and within the `Custom Social Connection` extension itself. In our example, we chose to also offer authentication via `Username-Password-Authentication`, `facebook` and `google-oauth2`, besides `Verimi`, thus the respective switches are enabled.

![Placeholder](img/app-connections.png)

In the end, the application is listed in the `Applications` overview:

![Placeholder](img/Screen_Shot_2019-04-02_at_10.20.09_AM.png)

### Auth0 Dashboard: Users

Looking at the user list and into a user's detail after he authenticated:

![Placeholder](img/Screen_Shot_2019-04-02_at_10.21.15_AM.png)

The user's primary identity provider is shown as `oauth2`...

![Placeholder](img/Screen_Shot_2019-04-02_at_10.21.25_AM.png)

... and the raw json of this user contains the Verimi identity of the user:

![Placeholder](img/Screen_Shot_2019-04-02_at_10.22.01_AM.png)


## Verimi Backend

After logging into the Verimi backend as a user, you see the dashboard...

![Placeholder](img/Screen_Shot_2019-04-02_at_10.26.58_AM.png)

...with further options to look at your data...

![Placeholder](img/Screen_Shot_2019-04-02_at_10.27.13_AM.png)

...as well as all your recent activities:

![Placeholder](img/Screen_Shot_2019-04-02_at_10.41.14_AM.png)

## Photo Credits
[Photo by Fezbot2000 on Unsplash](https://unsplash.com/photos/V4CU3OBsrEk)
