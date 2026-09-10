# My Input

### <u>CPU Util</u>

We underestimate how much cpu utilization and core utilization helps in increasing how well our server works. Utilizing the resources of our server to the fullest is extremely important, especially under load. Instead of running our nodejs app on a single core and using just that core and leaving the rest of our cores idle, we can spin it up on the all the cores and run one instance of the server on each core. This brings the idle time to almost zero and cpu usage to almost 100%, under load. Then we can run test to see our limits under this condition and works towards doing other things to improve it.

Node is single threaded(`./playground/single-thread.js`). So running a single instance runs that thread on single core, nodejs has its way of utilizing the remaining cores with its libuv workers but still won't give us the perfomance we want. We can't do two things at once but we can make it look like so by using things like async and await. But to fully utilize our cores, we can spawn multiple threads accross our cores as in `./playground/multi-thread.js`. It is harder to deal with and more complicated things to handle but it allows us to do more than one thing at a time. we can utilize our cpu to the fullest and drop our idle time to almost zero by doing this.

We can also do this by clustering with pm2, which i have implemented before using ecosystem.config


### <u>Framework overhead</u>

On small servers handling small amount of requests, the overhead being introduced by the framework doesn't matter much. But when we are handling thousands of requests, the overhead added by the frameworks start to compound. 

In our case, Express is quite slow. so it makes sense to chevk for other alternatives that are faster. For example, on Fastify docs, they claim to be at least 3x faster than Express, which is something. An endpoint that handles about 20,000rps on express would handle about 60,000rps on fastify, which is quite a leap and matters when you are handling large amount of requests. Imagine aiming for 600K and the endpoint currently handles 200K, simply eliminating the framework overhead puts you well above your goal. There is another one as fast or even faster than fastify which is cpeak, and the syntax is exactly the same with express so its a safer alternative.

<i>run test for nestjs later.</i>

A language closer to the computer like c++ or c is even faster. So i figured the overhead for something like assembly will be even faster. We only tested one other alternative with is [drogon(cpp)](https%3A%2F%2Fgithub.com%2Fagile8118%2Fcpp-1m-rps). 


### <u>Machine Limitations</u>

Size, Ram, throughput, iops, network speed..

Simply moving to a larger machine improves things...obviously but it's not that straightforward. I'll go into details in otyher sections because there's a chance we keep on burning money to upgrade machine for a subpar perfomance. I think i'll go through all the things i listed above.

### <u>Network speed</u>

Network speed is extremely important in large amount of requests. Each request size might look small even with a number of data in the request and in the response, might still not be more than a few KB. Lets say a server has the ability to process 5GB/s or 40Gb/s. This is quite huge and bigger than what my own personal PC can handle. But if we have request that just 10KB(could be higher/lower depending on the request and response size) and we expect our server to handle just 100,000rps. That's about 0.95GB/s which will work great because it is well below our spec. But imagine having to handle 1 Milliion rps for that same size, that requires a network bandwidth of about 10GB/s, which goes far beyond what our server can handle, so we expect to get a lot of errors...You could upgrade the machine, But it's expensive so i suggest upgrading to a machine with higher network perfomacne should be after you've exhausted other options and the other parts need upgrades.

In fact, sometimes, launching your own computer is cheaper than buying servers online if you need a network bandwidth that can handle 1M requests per seconds.

And instead of just aiming for a higher network bandwidth, we could also just deploy new servers and use load balances. we'll discuss this more in that section.

Just note that this is one of the limitations where you might actually start scaling horizontally instead of vertically as it's stupidly expensive to keep scaling vertically.


### <u>Optimize Database Queries</u>

Using a seperate large machine on the same private network goes a long way in increasing the rps. Instead of just using a db on the same machine as the server. Now, the amount request the db machine can handle greatly affects what our server can handle, especially for requests that have database operations in them. So yh, get a large comfortable machine for your db. and don't forget to set it up so you can use it to its full potential, increase the connection count, sessions count, iops, cluster mode, the pool, the network sppeed, etc etc

Optimize your queries and test them at scale. as obvious as this might sound, it is the bottleneck for a lot of servers. some queries might be extremely fast the table has just 100 entries, but what if it has a million? 2 million? will it still be fast? is there something you can do to increase the speed? This is not just about queries that work, this is about queires that will still work fast in large svale.

### <u>Leverage Memory(RAM, Redis)</u>

The RAM is aboout 10x faster than the disk storage to read/write and it's access time is 1000x faster. So leverage your ram to store some things so user get responses fast and offload them to your disk storage in the background. Of course, it adds a layer of complexity but you'd be pleasantly surprised at how fast it makes things.

The easiest way and the one i;ve implemented before is using redis and a queue. And it helps if your RAM is monstrously large too. 

Some ednpoints can do this method and others can write to disk directly. it's nice to leverage redis, it greatly improves the perfomance of your app.

You can also run redis in cluster mode to use more of cpu power and increase the available resources.


### <u>Horizontal/Vertical scaling</u>

For one reason or the other we might hit the limit of our machine and our machine won't be able to handle more than a level of requests, at that point, we might upgrade our machine and add or the spec. That's what it means to scale vertically. Which is fine, at some points, but it genuinely gets to a point where it jsut doesn't cut it anymore, you keep on upgrading and the server simply doesn't improve much. Or maybe just as design, you scale vertically then. That simply means setting up another machine with the same server and using a load balancer to share traffic between both machnes.


##### Will add more when i remember.



### Setup

To be able to run the code, you need to have Node.js, Redis, and Postgres installed.

Clone the repository and create a file in the database directory called `keys.js` and put the following content in there:

```javascript
/**
 * If you have a freshly installed Postgres with the default config,
 * these should work even if you don't have the benchmark database
 * created. But if you have changed anything, like adding a password,
 * please make sure to specify the correct value.
 */
const keys = {
  dbUser: "<your-postgres-username>",
  dbHost: "localhost",
  dbDatabase: "benchmark",
  dbPassword: "",
  dbPort: 5432,
};

export default keys;
```

Once done, run this command from the root directory to install the dependencies:

```
npm install
```

Then, initialize the Postgres database by running:

```
npm run seed
```

Now, you can run either the Express.js, Fastify, or Cpeak version (all have the same logic). They are just 3 different frameworks for Node.js.

```
node cpeak.js
```

or `node express.js` or `node fastify.js`.

Then you should get a log like this:

```
Cpeak server running at http://localhost:3000
[redis] standalone ready.
[postgres] connected successfully to benchmark.
```

**Redis cluster mode**: If you want to run the app with Redis in cluster mode, use the [redis.sh](redis.sh) file to set it up.

---

### Environment Variables

You can customize how the application runs by passing these 2 environment variables:

- `REDIS_CLUSTER` to indicate whether the app should connect to Redis cluster mode or not. Default: "false"
- `PG_CONNECT` to indicate whether the app should connect to Postgres database. Default: "true"

Example:

```
PG_CONNECT=false REDIS_CLUSTER=true node express.js
```

Output should be:

```
Express server running at http://localhost:3001
[redis] cluster ready. Total nodes 30 (masters: 15, replicas: 15)
```

---

### Node.js Cluster Mode

If you want to run the app in cluster mode, you can use PM2 for it. Check first if you have pm2 installed by running `pm2 --version` and if you don't have it, install it by running `npm install -g pm2`.

Then you can start the app in cluster mode by running:

```
pm2 start ecosystem.config.cjs
```

Check the `ecosystem.config.cjs` to change the [environment variables](#environment-variables).

The above command will run the cpeak server by default. If you want it to run Express or Fastify instead, specify the F environment variable like this:

```
F=express pm2 start ecosystem.config.cjs
```

or `F=fastify pm2 start ecosystem.config.cjs` to start the Fastify server.

You can run `pm2 logs` to check the logs of the application when running in cluster mode.

---

### Other Commands

When seeding the database, specify the -r option to indicate how many records should be added to the database:

```
npm run seed -- -r 20000000
```

_This will insert 20 million records into the codes table._

To move all the Postgres data over to Redis, run:

```
npm run migrate
```

_This does not work with Redis in cluster mode._
