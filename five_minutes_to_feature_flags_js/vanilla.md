We're starting off with a "Hello, World" server, stored in `01_vanilla.js`. You can use the Editor tab (up at the top left of the terminal on the right) to view that file, or just click on the command below
to print the file out to the terminal:

```
cat ~/app/01_vanilla.js
```{{exec}}

This is pretty much the most basic express server you can imagine - a single endpoint at `/`{{}} that returns a plaintext `"Hello, world!"`{{}} response.

Start the server by clicking the command below:
```
node ~/app/01_vanilla.js
```{{exec}}

You should be informed of a `Server running on port 3333`.

Now you can [visit this actual running server in a browser]({{TRAFFIC_HOST1_3333}}) to see its output. 

Alternatively you can be old-school and test its responses from the command line. open a new terminal Tab (click `+`{{}} next to `Tab 1`{{}}) then click the following command:

```
curl http://localhost:3333
```{{exec}}

Either way, you should see a very vanilla
```
Hello, World!
```

## With cows, please
Let's imagine that we're adding a new, experimental feature to this hello world service. We're going to upgrade the format of the server's response, using cowsay!

However, we're not 100% sure that this cowsay formatting is going to work out, so for now we'll protect it behind a conditional:

```javascript
import 'cowsay'
...
routes.get('/', async (req, res) => {
  // set this to true to test our new
  // cow-based greeting system
  const withCow = false
  if(withCow){
    res.send(cowsay.say({text:'Hello, world!'}))
  }else{
    res.send("Hello, world!")
  }
})
```{{}}

You can view the full file (`02_basic_flags.js`) in the Editor tab, or just print it out by clicking below:

```
cat ~/app/02_basic_flags.js
```{{exec interrupt}}

Let's run this new version of the server.  Flick back to tab 1 and try out the new code:

```
node ~/app/02_basic_flags.js
```{{exec interrupt}}

Back to tab 2 to re-curl the server (or [load it in the browser]({{TRAFFIC_HOST1_3333}})):

```
curl http://localhost:3333
```{{exec}}

Don't be surprised if the output looks the same as before. By default our service continues to work exactly as it did before, because we have that `withCow` value hardcoded to `false`.

But if we update it to `true`, our output should change. Give it a try if you want: editing the file in the IDE, restart the server, and check the response.

You'll want to update `app/02_basic_flags.js`, this line here:

```javascript{4}
routes.get('/', async (req, res) => {
  // set this to true to test our new
  // cow-based greeting system
  const withCow = true
  if(withCow){
    res.send(cowsay.say({text:'Hello, world!'}))
  }else{
    res.send("Hello, world!")
  }
})
```

now if we restart the server:
```
node ~/app/02_basic_flags.js
```{{exec interrupt}}


Then [our server's response]({{TRAFFIC_HOST1_3333}})

```
$> curl http://localhost:3333
```{{exec}}

should look a bit more exciting:

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

# The Crudest Flag
That `withCow`{{}} boolean and its accompanying conditional check are a very basic feature flag - they let us hide an experimental or unfinished feature, but also easily switch the feature on while we're building and testing it.
