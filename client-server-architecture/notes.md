# Client-Server Architecture

Every application you use — a website, a mobile app, a game — runs on some version of client-server architecture. It’s the answer to one basic question: who does the work, and where does it happen? These notes walk through that idea from its simplest form (everything on one machine) to how real, large-scale systems are actually built today.

## What it is

A client sends a request. A server processes it and sends back a response.

Example: when a client visits a website (say, google.com), the client’s browser sends a request, and the server hosting google.com processes it and returns the results.

---

## How it works

When you type a URL, four things happen before you see a page:
1. **DNS (Domain Name System)** — converts the domain name into an IP address.
2. **TCP/IP** — the protocol that handles data packets being transferred to the server.
3. **Port** — the server address number for a specific service (e.g. port 80 for a website).

All of this happens over the internet, following the HTTP/HTTPS protocol for communication.

*Refer to the image here: `./diagrams/01-how-it-works.svg`*

---

## Types of Client-Server Architecture

### 1. 1-Tier (Monolithic)

Everything — client, server, data, all of it — resides at the same level, handled on the same machine.

Example: MS Excel.

*Refer to the image here: `./diagrams/02-1-tier.svg`*

**Pros**
* Easy to handle and manage
* Good for offline applications and small ones

**Cons**
* Difficult to scale
* No separation of concerns

---

### 2. 2-Tier Architecture

Client and server are separate. Two common variants:
* **Fat client, thin server** — client holds UI + business logic, server is just a database (older enterprise style)
* **Thin client, fat server** — client only renders UI, server holds the logic and talks to the DB (most modern web apps)

*Refer to the image here: `./diagrams/03-2-tier.svg`*

**Pros**
* Simple and fast for a small number of users

**Cons**
* State/scaling problems when clients increase
* Performance issues as load grows
*(Fix: solved later by adding a load balancer + multiple servers, not one bigger server.)*

---

### 3. 3-Tier Architecture

Here the client-server duo becomes three layers, each handling its own job:
* **Presentation layer** — the client, what the user sees
* **Business/Application layer** — the logic, processing, preventing bad requests
* **Data layer** — storage of data/entities

*Refer to the image here: `./diagrams/04-3-tier.svg`*

**Important:** the 3 tiers are these 3 layers, not literally “client machine, server machine, database machine.” A common deployment does put each layer on its own machine — but that’s one deployment choice, not the definition. The client is presentation; the app server is business logic; the DB is data — but they can also be combined differently.

**Pros**
* **Scalability** — client, server, DB can scale independently by adding more resources
* **Security** — sensitive business logic doesn’t sit exposed on the client

**Cons**
* **Single point of failure** — if control flows through one node, it can crash the whole app
* **Performance bottlenecks** — all traffic hitting the same server can slow things down as it scales

---

### 4. N-Tier Architecture

Has dedicated layers for each concern: presentation, business logic, caching, security, and more — separated out instead of bundled into one “server.”

*Refer to the image here: `./diagrams/05-n-tier.svg`*

**Pros**
* Easy to scale further (add more of any one layer without touching the rest)
* Strong security and separation between layers

**Cons**
* Complexity increases — more layers means more to build and manage
* Harder to debug when something goes wrong, since a request crosses more hops

What the “dedicated layers” actually are, in practice:
* **Load Balancer** — spreads incoming traffic across multiple servers
* **API Gateway** — single entry point for clients, routes requests to the right service
* **Cache (Redis / CDN)** — serves repeat data fast without hitting the database every time
* **Message Queue** — lets services hand off work to each other without waiting

**Why statelessness matters:** HTTP is stateless — the server doesn’t remember a client between requests. This is why load balancing works at all: since the server has no memory of you, your next request can go to a completely different server without breaking anything. The trade-off is that every request has to carry its own context.

---

## Session Management

Since HTTP is stateless, “staying logged in” needs a separate fix — the server isn’t remembering you on its own.
* The client sends a cookie or token (e.g. JWT) with every request.
* The server checks that token each time and identifies the user from it — it never relies on memory.

This is what makes stateless servers workable in practice: the state lives in the token, not in the server.

---

## Quick reference

| Situation | Use |
|---|---|
| Offline tool, one user | 1-Tier |
| Small app, few users | 2-Tier |
| Standard web app | 3-Tier |
| Large-scale system | N-Tier |
