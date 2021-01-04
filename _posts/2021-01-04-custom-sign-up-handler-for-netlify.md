---
layout: post
title: Custom Sign Up Handler for Netlify
date: 2021-01-04 13:34 -0800
tags: netlify, serverless, auth
---

I've started getting back into serverless engineering recently and decided to give [Netlify](https://netlify.com) a try. I appreciate their focus on JAMStack and the availability of server-side functions that integrate fairly well into my local dev model.

One of the first tasks I tackled was integrating authentication with my backend. Netlify provides an Identity service which handles registration, confirmation, and log in. It also supports custom metadata which I wanted to use to tie the DB-side ID with the netlify identity but ran into problems. I tried to use [Netlify's built-in identity hooks](https://docs.netlify.com/functions/functions-and-identity/) but wasn't able to update metadata as they describe so I created a custom solution.

## Creating the Sign Up Form

Netlify provides a couple of drop-in libraries but I needed to create a form to trigger my custom logic. Here's the simplified version. The notable entries are the form `action` and the hidden `to` field. The `action` is my custom sign-up function and the `to` field configures the HTTP redirect after the sign-up completes.

```jsx
<form action="/api/sign-up" method="POST">
<input type="hidden" name="to" value="/sign-up/confirm" />
<input placeholder="name" name="name" required />
<input placeholder="email" name="email" required />
<input
	placeholder="password"
	type="password"
	name="password"
	required
/>
<input type="submit" />
</form>
```

## Redirecting the API

Next I needed to connect the `action` above to the backend Netlify function. This step isn't necessary but doesn't provide a useful abstraction to decouple the implementation of sign-up from the front end. Adding a redirect to the `netlify.toml` file in the root of the project did the trick. I used a catch-all for any future API methods but you could limit it if you wish.

```toml
  [[redirects]]
    from = "/api/*"
    to = "/.netlify/functions/:splat"
    status = 200
```

## Handling the request

> Note: This code is run through `netlify-lambda build functions` to generate a bundle that is compatible with Netlify functions. More info on the [netlify-lambda GitHub repo](https://github.com/netlify/netlify-lambda).

To create a user from a netlify function, you have to pull in `gotrue-js`, the client for the Identity service. I found `node-fetch` was also required to patch in `fetch` to `global` which `gotrue-js` attempts to use.

My process was to create the user in the database first to obtain the `id` and then pass that `id` along to the third arg of `signup()` to include it in the `user_metadata`. You'll likely want a little more transaction management since it's possible that `signup()` might fail and you'll need to clean up the database. This could also happen in the email confirmation phase. Once both the database and identity are created, I redirect to the desired page.

At the moment, this code doesn't log the user in. I plan to add that next so they can start using at least some of the app before confirming their email. However, Identity will block the user from logging in until they confirm as far as I can tell.

```js
// functions/sign-up.js
import GoTrue from "gotrue-js";
import fetch from "node-fetch";
import qs from "querystring";

import { register } from "../service/auth";

async function signUp(event, context) {
  const { origin, referer } = event.headers;

  // Path fetch which is required by GoTrue but wasn't available via netlify dev
  global.fetch = global.fetch || fetch;

  try {
	// Initialize GoTrue (Netlify's identity service API)
    const auth = new GoTrue({
      APIUrl: `${origin}/.netlify/identity`,
      audience: "",
      setCookie: false,
    });

    // Decode the form parameters
    const { name, email, password, to } = qs.decode(event.body);

    // register the user in your database and extract the ID
    const {
      ref: { id },
    } = await register(name, email);

    // Sign up the user with Netlify and include the id in the user metadata
    await auth.signup(email, password, { name, id });

    // Determine the redirect URL: Either provided by the to field or returning to the referring URL
    let Location = to;
    if (!Location) {
      const url = new URL(referer);
      Location = url.pathname + url.search + url.hash;
    }

    // Return the redirect
    return {
      statusCode: 303,
      body: "User created",
      headers: {
        Location,
      },
    };
  } catch (e) {
    return {
      statusCode: 500,
      body: "Unknown Server Error",
    };
  }
}

export { signUp as handler };
```