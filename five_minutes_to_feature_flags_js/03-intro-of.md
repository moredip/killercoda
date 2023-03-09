Managing these flags by changing hardcoded constants gets old fast though. A team that uses feature flags in any significant way soon reaches for a feature flagging framework. Let's move in that direction by setting up the [OpenFeature](https://openfeature.dev) SDK:


We'll create a new version of our server (`app/03_openfeature.js`) which looks like this:

```javascript{6}
import { OpenFeature } from '@openfeature/js-sdk'

const featureFlags = OpenFeature.getClient()

routes.get('/', async (req, res) => {
  const withCows = await featureFlags.getBooleanValue('with-cows', false)
  if(withCows){
    res.send(cowsay.say({text:'Hello, world!'}))
  }else{
    res.send("Hello, world!")
  }
})
```

We're importing the `@openfeature/js-sdk`{{}} npm module, and using it to create an OpenFeature client called `featureFlags`{{}}. We then call `getBooleanValue`{{}} on that client to find out if the `with-cows` feature flag is `true` or `false`, and as before we either show the new cow-based output or the traditional plaintext format depending on whether `withCows` is true or false.

The big difference is that rather than using a hard-coded conditional we're not asking for the state of the flag dynamically, at runtime, using `getBooleanValue()`.

Let's try this new server out. Head back to Tab 1 and run it:

```
node ~/app/03_openfeature.js
```{{exec interrupt}}

Over in tab 2 we can re-curl the server (or [load it in the browser]({{TRAFFIC_HOST1_3333}})):

```
curl http://localhost:3333
```{{exec}}

and we should be back to getting a vanilla `Hello, World!` response. Why is that?

Well, the OpenFeature SDK doesn't provide feature flagging capabilities by itself. We have to configure it with a "[provider](https://docs.openfeature.dev/docs/specification/glossary/#provider)" which connects the SDK to a feature flagging implementation which can actually make the flagging decisions we need. (You can read more about OpenFeature's architecture [here](https://docs.openfeature.dev/docs/reference/intro#what-is-openfeature).)

Since we haven't configured the SDK with a provider it has no way of making feature flagging decisions and will just return default values. In this case, `with-cows` is defaulting to `false`, so now we don't see any cows in our output.

Let's fix that by configuring the SDK with a feature flag provider!

## Configuring OpenFeature

If this was a fancy production-grade system we'd probably want to connect the OpenFeature SDK to a full-fledged feature flagging system - a commercial product such as LaunchDarkly or Split, an open-source system like [FlagD](https://github.com/open-feature/flagd), or perhaps a custom internal system - so that it can provide flagging decisions from that system.

Connecting OpenFeature to one of these backends is very straightforward, but it does require that we have an actual flagging framework set up. For now, just to get started, we'll just configure a really, really simple provider that doesn't need a backend. 

We can see what that configuration looks like in yet another version of our server, `04_openfeature_with_provider.js`. Here's the relevant part of that file:

```
import { MinimalistProvider } from '@moredip/openfeature-minimalist-provider'

const FLAG_CONFIGURATION = {
  'with-cows': true
}

const featureFlagProvider = new MinimalistProvider(FLAG_CONFIGURATION)

OpenFeature.setProvider(featureFlagProvider)
const featureFlags = OpenFeature.getClient()
```{{}}

This minimalist provider is exactly that - you give it a hard-coded set of feature flag values, and it provides those values via the OpenFeature SDK.

In our `FLAG_CONFIGURATION`{{}} above we've hard-coded that `with-cows`{{}} feature flag to `true`{{}}, which means that conditional predicate in our express app will now evaluate to true, which means that our service should now start providing bovine output. Let's check!

First, we'll start up this version of the server. Head back to Tab 1 and launch it:

```
node ~/app/04_openfeature_with_provider.js
```{{exec interrupt}}

Now in tab 2 we can re-curl the server (or [load it in the browser]({{TRAFFIC_HOST1_3333}})):

```
curl http://localhost:3333
```{{exec}}

The output should look like this:

```
 _______________
< Hello, world! >
 ---------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```{{}}

Our feature flagging system is working! Our server asks OpenFeature for the state of the `with-cows` flag, specifying a default of `false`. The OpenFeature SDK passes this request on to the configured provider, which returns a value of `true`, enabling the flag and thus the new and improved formatting.

#### flipping a flag

We can double-check that this is what's happening by updating the provider's flag configuration. Open up `04_openfeature_with_provider.js` in the Editor tab, and update the configuration to turn off the `with-cows` feature:

```javascript{3}
const featureFlags = OpenFeature.getClient()
const FLAG_CONFIGURATION = {
  'with-cows': false
}

const featureFlagProvider = new MinimalistProvider(FLAG_CONFIGURATION)
```

Now hop over to tab 1 and re-start the server:

```
node ~/app/04_openfeature_with_provider.js
```{{exec interrupt}}

and over in tab 2 reload the response (or [load it in the browser]({{TRAFFIC_HOST1_3333}})):

```
curl http://localhost:3333
```{{exec}}

You should see that we're back to the cow-less, vanilla `Hello, World!` response. Feature flags in action!

This is all very exciting, but we're still having to make changes in source code to control that feature flag, due to the limitation of
this extremely basic in-memory provider. Next we'll look at upgrading to a *real* feature flagging backend...
