# Latency vs Throughput vs Bandwidth: The Simple Mental Model Every Engineer Should Know

Latency, throughput, and bandwidth come up in almost every conversation about system performance. They sound alike, people use them interchangeably, and they measure three very different things. Mixing them up leads to fixing the wrong problem.

What clears it up is a single question: **is it slow for one user, or only when many users are using it?** The answer tells you which problem you have and what to fix.

## The Real-Life Example

Imagine a small pizza shop with 2 ovens.

*   **Latency** is how long one customer waits for their pizza. Say 10 minutes.
*   **Throughput** is how many pizzas the shop actually makes per hour. Say 8 on a normal day.
*   **Bandwidth** is how many pizzas the shop could make per hour if everything ran perfectly. Say 12, because each of the 2 ovens bakes 6 pizzas an hour.

*Refer to the image here: `./diagrams/01-pizza-shop-bandwidth.png`*

These are three different questions: how long for one, how many in total, and what’s the maximum possible. Throughput can never be higher than the maximum, and in practice, it’s lower.

## Watch it Happen: Quiet Evening vs. Friday Rush

Here is the same shop on two different days. The shop and the ovens are identical. Only the number of customers changes.

*Refer to the image here: `./diagrams/02-friday-rush.png`*

On the quiet evening, a customer arrives every 12 minutes. There is never a line, and everyone waits exactly 10 minutes. 

On the Friday rush, a customer arrives every 2 minutes. The ovens can’t keep up, so the line grows, and the wait climbs from 10 minutes to 40 and beyond.

Notice that **no single pizza got slower**. Each one still takes 10 minutes to bake. The extra waiting comes entirely from the line. 

> [!IMPORTANT]
> This is the core idea: when a system can’t handle the volume (a throughput limit), a queue builds, and latency goes up for everyone in it.

## The Question That Tells Them Apart

When someone says “this is slow,” ask: **is it slow for one customer, or only when many customers arrive?**

*   **Latency Problem:** If one customer walks into an empty shop and their pizza still takes 40 minutes, the shop isn’t short on capacity. Making that one pizza is slow. 
*   **Throughput Problem:** If the shop is fine with a few customers but the line stretches out the door when hundreds arrive, the shop can’t keep up.

### Why "just buy more ovens" doesn't always work

More ovens mean more pizzas per hour, so the line shrinks. That’s throughput improving. But one pizza still takes 10 minutes. More ovens never make a single pizza faster.

There’s also a catch. If all the ovens share one dough machine, buying more ovens does nothing, because the dough machine is the real limit.

---

## The Same Thing Inside a Computer

Now let’s move to software. Imagine a food delivery app where you open a restaurant’s page.

The ideas stay the same, only the words change. A customer’s wait becomes the time for one request (**latency**), pizzas per hour becomes requests handled per second (**throughput**), and what the ovens could make becomes the system’s maximum capacity (**bandwidth**).

> [!NOTE] 
> **Bandwidth vs Throughput:** Bandwidth is the maximum a connection can carry, like a 1 Gbps internet plan. Throughput is what you actually get, maybe 600 Mbps. Maximum versus actual.

### Latency in a Computer: Where does one request spend its time?

Say the restaurant page takes 3 seconds to open, even at 3 AM when almost nobody is online. Nobody else is using the system, so this is a latency problem. To fix it, trace one request and see where the time goes.

*Refer to the image here: `./diagrams/03-latency-computer.png`*

In this example, almost all the delay comes from one slow database query. Adding more servers would not help here, because one request would still wait 2.4 seconds on that query. 

**The fix is inside the request:** add an index, rewrite the query, or cache the result. This is why you measure before you fix. The slow part is rarely where you first guess.

### Throughput in a Computer: When many users arrive

Now take the same app on a Friday evening. Say you have 5 servers, and the database behind them can handle 500 queries per second. Watch what happens when traffic goes from 200 requests per second to 1,000.

*Refer to the image here: `./diagrams/04-throughput-computer.png`*

During the quiet period, the database keeps up easily and every response takes about 20 ms. During the rush, requests arrive faster than the database can finish them. A queue builds, response time climbs past a second, and eventually requests start timing out. 

The servers aren’t the problem. They’re not overloaded. The database is. It’s the dough machine from the pizza shop: the one shared part that limits everything. Adding a sixth server would just send more requests to the same full database.

**The real fix is to give the database less work:** 
- Save popular answers so they don’t need a database lookup every time (**caching**)
- Spread the reading across extra copies of the database (**read replicas**)
- Save data in groups instead of one at a time (**batching**)

So the complete answer is: add more servers if you need to, but first find the part that is actually the limit.

## Three Mistakes to Avoid

1. **Using throughput and bandwidth as if they mean the same thing.** Bandwidth is the maximum capacity. Throughput is what you actually achieve. Throughput is almost always lower.
2. **Adding servers to fix a latency problem.** More servers help when many requests arrive together. They don’t make one request faster.
3. **Looking only at the average.** An average can hide slow users. Imagine 99 customers get their pizza in 5 minutes, but 1 customer waits 60 minutes. The average still looks great, yet one person had a terrible experience.

> [!TIP]
> This is why engineers look at the slowest few percent instead using percentiles like **p99**. "p99" simply means: 99 out of 100 requests were faster than this number. If your p99 is high, a small group of users is having a bad time, even when the average looks fine.

If you remember one thing, make it this question: **is it slow for one request, or only under load?**
