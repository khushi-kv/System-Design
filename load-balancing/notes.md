# Load Balancing

Basic problem: one server can't handle everything forever. Once you have more than one server, something has to decide which request goes where. That "something" is a load balancing algorithm.

There isn't one correct algorithm — each one is just a different trade-off between how much it checks before deciding and how much effort that checking costs. Simple algorithms decide fast but can get it wrong. Smarter ones decide better but cost more to run.

Going through them roughly in the order it made sense to learn them.

---

## 1. Round Robin

The simplest one. LB keeps a pointer to the list of servers, and every new request just goes to whoever's next in line. After the last server, it loops back to the first.

*Refer to the image here: `./diagrams/01-round-robin.svg`*

So with servers A, B, C — request 1 → A, request 2 → B, request 3 → C, request 4 → back to A. No server is checked for how busy it actually is, it's purely a rotation.

**Downside:** assumes every server can handle the same load, which isn't usually true. One server could be under more strain than the others and it'll still get its turn on schedule.

---

## 2. Weighted Round Robin

Same rotation idea, but now each server gets a weight based on its capacity, and gets that many turns instead of just one.

*Refer to the image here: `./diagrams/02-weighted-round-robin.svg`*

Weights 4:2:1 for A:B:C means A gets 4 out of every 7 requests, B gets 2, C gets 1. Still a fixed rotation underneath, just uneven turns now.

**Downside:** the weight is set once (usually based on server specs) and never re-checked. Doesn't account for what's actually happening on the server right now.

---

## 3. Least Connections

First one that actually looks at the servers before picking. Checks how many connections each server currently has open, sends the request to whoever has the fewest.

*Refer to the image here: `./diagrams/03-least-connections.svg`*

Works well when connections stay open for very different lengths of time — websockets being the classic case. A server holding a few long-lived connections is doing more work than the number "3" would suggest, and this algorithm actually accounts for that.

**Downside:** it's still just counting connections, not measuring how heavy each one actually is. Two servers can both show "10 connections" and be in completely different states of strain.

---

## 4. Weighted Least Connections

Combines the last two. Instead of comparing raw connection counts, it compares connections ÷ weight — so a server's load is judged relative to what it can actually handle.

*Refer to the image here: `./diagrams/04-weighted-least-connections.svg`*

Quick example — A: 16 connections, weight 4 → ratio 4. B: 6 connections, weight 2 → ratio 3. C: 4 connections, weight 1 → ratio 4. B wins even though A and C both look "less loaded" on paper by raw numbers.

**Downside:** same blind spot as least connections (a connection isn't a real measure of work), plus now it also depends on the weights being set correctly.

---

## 5. Least Response Time

Stops looking at connection counts, looks at how fast each server has actually been responding recently, and sends the request there. In practice usually paired with connection count too, so a server doesn't get flooded just for having one lucky fast reply.

*Refer to the image here: `./diagrams/05-least-response-time.svg`*

**Downside:** has to keep tracking response times continuously, which is overhead. And it's reacting to the recent past, not the current second — so if traffic spikes suddenly, it can lag behind reality for a bit.

---

## 6. Power of Two Choices

For when checking every server is itself too expensive — think hundreds of servers, checking all of them for every single request adds up fast.

*Refer to the image here: `./diagrams/06-power-of-two-choices.svg`*

Instead of checking everyone, it picks just two servers at random, compares those two, and sends the request to whichever of the two is less loaded. Sounds like it shouldn't work well since most servers are ignored, but the math actually holds up — comparing two random picks against each other avoids the worst-loaded servers most of the time, for way less overhead than checking the whole pool.

**Downside:** it's probabilistic, not guaranteed — bad luck can still pick two heavily loaded servers. Also only really pays off with a large pool; on a small number of servers, plain least connections does the job for less complexity.

---

## 7. IP Hash

Different goal from everything above — here we actually want the same client landing on the same server every time, not spread out. Useful when a server is holding onto session data or a warm cache for that specific client.

*Refer to the image here: `./diagrams/07-ip-hash.svg`*

Mechanism: hash the client's IP, take that hash mod (number of servers), and that gives a fixed index. Same client → same index → same server, as long as the server count doesn't change.

**Downside:** "as long as server count doesn't change" is the catch. Add or remove one server and the mod value changes for basically everyone, so most clients get reshuffled to a different server — right when you probably scaled up to reduce pressure, not add chaos.

---

## 8. Consistent Hashing

This exists specifically to fix IP hash's weak spot. Important to get this right — it's not "a better modulo," it works because it skips modulo across the whole pool entirely.

*Refer to the image here: `./diagrams/08-consistent-hashing.svg`*

Both servers and keys (clients/requests) are placed on a hash ring. A key belongs to whichever server is next, going clockwise, from where it lands on the ring. When a server is added, only the keys sitting right before it on the ring need to move — everything else stays put.

Compare that to IP hash: adding one server there can reshuffle ~80% of clients because the mod value changed for everyone. On a ring, adding one server only disturbs the keys nearest to it.

One detail that's easy to skip — real setups don't place each server on the ring just once, they use **virtual nodes**: each physical server gets mapped to many points around the ring. Without this, a ring with few servers can end up lopsided, with one server owning way more of the ring than another.

**Downside:** more complex to build and reason about than a plain modulo hash. Only worth it if the server pool actually resizes often — for a pool that barely changes, IP hash does the same job for less effort.

---

## Quick reference

| Situation | Use |
|---|---|
| Servers are identical, nothing fancy needed | Round Robin |
| Servers differ in capacity | Weighted Round Robin |
| Connections stay open for very different durations | Least Connections |
| Same as above + servers differ in capacity | Weighted Least Connections |
| User-facing latency is the priority | Least Response Time |
| Pool is huge, checking everyone is too expensive | Power of Two Choices |
| Same client needs the same server, pool size is stable | IP Hash |
| Same as above but pool resizes often | Consistent Hashing |

Basic pattern across all of these: the less an algorithm checks, the faster/cheaper it is — but the more likely it is to get it wrong when things don't match what it assumed. Check less → fast, but more room for mistakes. Check more → accurate, but slower and costs more.