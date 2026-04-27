### Overview

AutoShares is a cross-platform online trading solution for Stocks, Options, and ETFs with all-encompassing trading functionality. Even though AutoShares provides both the web and the mobile interfaces for executing trades and analyzing markets, some users opt to implement their own custom interfaces that suit their requirements. For this purpose, AutoShares offers a trading platform as well as extended REST API that can be invoked to perform various trading operations like creating users, getting quotes, and placing orders. 

In this scenario AutoShares serves as the backend platform for your own custom-made web terminal or mobile app. Your task is to develop the mobile and web UI that invokes our API to perform all of the trading operations. On our side, we will execute your customers' orders on the required exchange, settle all transactions with the clearing firm, and take care of all of behind-the-scenes technicalities.

Another frequent use-case of our API is to augment the existing functionality of AutoShares trading platform by designing custom widgets. In this scenario you create your own custom JavaScript-based widgets that can load various information about the user's positions, place new orders, and perform just about any other action. For example, you can create a widget that displays the user's most frequently traded securities; or a widget that displays the list of users' positions with the highest profit or highest loss.

### Trading APIs

This is the trader's API that invokes actions typically performed by trader's widgets: placing orders, getting quotes, configuring price alerts, and so forth. This API can be called from anywhere simply using your app's API key and a user's credentials. 

---

# Working with the APIs

### Authentication

Every call to AutoShares API must include an API secret key. After you create an AutoShares account, we generate API keys for you. To start working with the API with your keys, you need to activate your account. 


### Adding User Accounts

This API endpoint enables you to bind a new or an existing trading account to an existing user.


### Sign Access Agreements

All users on the system must sign agreements. This GET endpoint enables you to fetch the list of all outstanding agreements that must be signed by the user.

### The Flow

In its most basic form, you will register your application with AutoShares, redirect a browser to a URL, parse a token from the responding redirect, and send the token to whichever AutoSharesAPI you wish to access.

### Authorization Codes

Authorization codes are short-lived (they expire in 10 minutes). A code is provided after an investor authorizes your application for access to their account. These codes are exchanged for access tokens which can be used to make API requests.

### Access Tokens

Access tokens (or Bearer tokens) are provided after a successful authorization code exchange. These tokens expire every 24 hours – no questions asked. Once a token expires you’ll need to exchange another authorization code to receive another access token.

### Refresh Tokens

Once refresh tokens are enabled for your application, you will receive the refresh token alongside every access token. You can exchange a refresh token for a new access token. Refresh tokens do not expire.

### Onboarding Accounts

Applicants must follow the workflow determined by the AutoShares implementation team. 

### Have Questions?

Our seasoned team of experts in brokerage operations and fintech are available to discuss your interest in connecting, integrating, and embedding functionality into your workflow using our API-based technology.

If you have any questions or need support, please don't hesitate to contact us at https://autoshares.com/support.

### Get Started, Today. 


