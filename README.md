# Shopify App Node + Memcached

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE.md)

This is a boilerplate app based on the official Shopify Node App to help speed up the initial setup. It adds [**Memcached**](https://memcached.org/downloads) as the storage method for Session and Shop data.

The default Shopify Node App does not persist the list of Shops or Session data, forcing a redirect back through the OAuth process everytime the app crashes or reloads during development.

It leverages the [Shopify API Library](https://github.com/Shopify/shopify-node-api) on the backend to create [an embedded app](https://shopify.dev/apps/tools/app-bridge/getting-started#embed-your-app-in-the-shopify-admin), and [Polaris](https://github.com/Shopify/polaris-react) and [App Bridge React](https://shopify.dev/tools/app-bridge/react-components) on the frontend.

This is a fork of the repository used when you create a new Node app with the [Shopify CLI](https://shopify.dev/apps/tools/cli).

## Requirements

- If you don’t have one, [create a Shopify partner account](https://partners.shopify.com/signup).
- If you don’t have one, [create a Development store](https://help.shopify.com/en/partners/dashboard/development-stores#create-a-development-store) where you can install and test your app.
- In the Partner dashboard, [create a new app](https://help.shopify.com/en/api/tools/partner-dashboard/your-apps#create-a-new-app). Check out the "Step-by-step" section for a screenshot walk through.
- [**Memcached**](https://memcached.org/downloads) running on default port 11211
- Ngrok for Local Development
- Git
- Node `>=` 16.13.0 (technically, only Polaris needs 16+, the server itself works on 14+ [Expect Breaking Changes from Shopify at anytime 😅](https://github.com/Shopify/polaris/issues/4549#issuecomment-959756118) )

## Installation

### Shopify Admin

You can either create a new App or connect to an existing App.

[Create New App](https://partners.shopify.com/apps/new/manual)

Simply name your App and use `https://localhost` for the URLs. These can be automatically updated with your Ngrok tunnel address by using the `shopify app serve` command.

<hr/>
<details>
<summary>Step-by-step Create App</summary>

### Step 1

![Employee data](http://imageurl "Employee Data title")

</details>
<hr/>

### Locally

Clone this repo:

```sh
git clone https://github.com/OffGridCode/shopify-app-node-memcached.git
```

Open the new directory:

```sh
cd ./shopify-app-node-memcached
```

Connect to the App you created:

```sh
shopify app connect
```

#### Local Development

`shopify app serve`

This will auto-magically update the App's URL settings. The App's root URL and following Allowed Redirect Urls will be added/updated:
<br>
`/auth/shopify/callback`
<br>
`/auth/callback`

#### Production

On your production server, rename `.env.example` to `.env` and update it with your App's API Keys:

```yaml
SHOPIFY_API_KEY={api key}           # Your API key
SHOPIFY_API_SECRET={api secret key} # Your API secret key
SCOPES={scopes}                     # Your app's required scopes, comma-separated
HOST={your app's host}              # Your app's host, without the protocol prefix

MEMCACHED=true                      # Enables or Disables Memcached, fallsback to in-memory
MEMCACHED_RETRIES=2
MEMCACHED_EXPIRES=2592000
MEMCACHED_USERNAME=
MEMCACHED_PASSWORD=
```

Don't forget to update your App's Shopify URL and Allowed Redirect settings to match your server/domain address.

<hr/>

And that's it!
You've now connected your local App to Shopify's embedded Admin section.

## FAQ

<details>
<summary><b>Why start with this template?</b></summary>

- Minimal changes to Official Shopify App
- Must work out of the box
- Fallback gracefully if Memcached fails to connect
- Maintain the latest major version of each Node module
- Up to date with original Shopify App code base
- Maintain the coding style of Shopify's developers (semicolons... semicolons everywhere...)
- Easily readable and modifiable code
- Use the most reliable Node modules available (Currently "memjs")
- If your App crashes, you won't lose Session or Shop data. Unless Memcached also crashes.. In which case it will redirect the user back through OAUTH process.
</details>

---

<details>
<summary><b>What is this Error?</b> <code>ECONNREFUSED 127.0.0.1:11211</code></summary>

```
MemJS: Server <localhost:11211> failed after (2) retries with error - connect ECONNREFUSED 127.0.0.1:11211
Memcached can't load Shops Error: connect ECONNREFUSED 127.0.0.1:11211
    at TCPConnectWrap.afterConnect [as oncomplete] (node:net:1187:16) {
  errno: -4078,
  code: 'ECONNREFUSED',
  syscall: 'connect',
  address: '127.0.0.1',
  port: 11211
}
```

Memcached isn't running. Start the memcached service and try again.
<br/>
If you don't have Memcached installed... You should probably install [**Memcached**](https://memcached.org/downloads)

</details>

---

<details>
<summary><b>What is Memcached?</b></summary>

[**Memcached**](https://memcached.org/downloads) is one of the fastest and most reliable way to store `{key:value}` data in-memory. Once you have Memcached running locally on the default port, everything should work smoothly.

</details>

---

<details>
<summary><b>What are some key changes to the official Shopify Node App?</b></summary>

Changed the default initialization of `SESSION_STORAGE` from `MemorySessionStorage()` to `CustomSessionStorage()`

```
Shopify.Context.initialize({
  ...,
  SESSION_STORAGE: new Shopify.Session.CustomSessionStorage(storeCallback, loadCallback, deleteCallback),
});
```

`/server/helpers/storage.js` exports `Memcached`, allowing you to `.set()`, `.get()`, and `.delete()` data from anywhere without reinitializing the client.

</details>

---

<details>
<summary></summary>

</details>

---

<details>
<summary></summary>

</details>

---

<details>
<summary><b>Can the API Tokens be used outside of the current Session?</b></summary>

You're thinking of "Offline" Tokens, which can be used to make REST or GraphQL calls to Shopify even if the user is no longer signed in. By default, Shopify's App only requests an "Online" token, but it can easily be modifed to request both.

</details>

---

<details>
<summary><b>Sooo how can I get an "Offline" Token</b></summary>

The OAuth process starts on line 14 of `/server/middleware/auth.js`. The method used is `Shopify.Auth.beginAuth(request, response, shop, redirectPath, isOnline)`.

Change `isOnline` to `false` and you'll get an "Offline" token as a response in the next step `/auth/callback` inside `session.accessToken`. Since you don't have an "Online" token at this point, the OAUTH process starts over and you'll be redirected back to `/auth` until `isOnline` is `true` and an "Online" Token is passed to the Session.

You can add some logic to check if the Shop doesn't exist in Memcached, request an "Offline" token, let the Shop save to Memcached, rerun `/auth`, and this time the Shop will exist so `isOnline` can be set to `true`, which will giv you an "Online" token and finish the Session process.

</details>

---

## Developer resources

- [Introduction to Shopify apps](https://shopify.dev/apps/getting-started)
  - [App authentication](https://shopify.dev/apps/auth)
- [Shopify CLI command reference](https://shopify.dev/apps/tools/cli/app)
- [Shopify API Library documentation](https://github.com/Shopify/shopify-node-api/tree/main/docs)

## License

This repository is available as open source under the terms of the [MIT License](https://opensource.org/licenses/MIT).
