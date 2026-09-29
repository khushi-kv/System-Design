# Forward Proxy vs Reverse Proxy

From the basic idea to the details you meet in real projects and interviews. Both sit in the middle and pass traffic along. The difference is whose side they are on.

*Refer to the image here: `./diagrams/01-proxy-middle.svg`*

When you open a website, your request doesn’t always go straight from your browser to the application server. Sometimes another server sits in the middle. What that server is called depends on who it works for.

## Forward proxy

Works for the client. Like an assistant who makes phone calls for you. The other side only sees the assistant’s number.

## Reverse proxy

Works for the servers. Like a company receptionist. You call one number and never know which employee handles you.

---

## How a forward proxy works

A forward proxy sits between the client and the internet. Instead of sending a request straight to the website, the client sends it to the proxy. The proxy passes it on, gets the answer, and hands it back.

*Refer to the image here: `./diagrams/02-forward-proxy.svg`*

### Why use a forward proxy

* **Privacy.** The website sees the proxy’s IP, not yours.
* **Access control.** A company can block certain sites or limit which outside services employees use.
* **Monitoring and logging.** All outgoing traffic passes through one place, so the company can record requests and enforce security rules.
* **Caching.** If many people want the same thing, the proxy keeps a copy and saves bandwidth.
* **Access through another network.** Some sites allow or block visitors based on location. A proxy in an allowed location makes the request for you, so the site treats the request as coming from there.

**Note:** Hiding your IP is not anonymity. The proxy owner can still see and log your traffic, and websites can still recognise you through logins, cookies and browser fingerprints. Only use proxies you trust.

---

## How a reverse proxy works

A reverse proxy sits in front of the backend servers. The client sends its request to it, usually without knowing a proxy exists. The proxy picks a backend, forwards the request, and returns the response. To the client, it looks like talking to a single server.

*Refer to the image here: `./diagrams/03-reverse-proxy.svg`*

### Why use a reverse proxy

* **Traffic management.** Spreads requests across many servers so none gets overloaded.
* **TLS termination.** The proxy handles HTTPS encryption and certificates in one place, so each backend doesn’t have to.
* **Caching.** Frequently requested responses are served from a copy, which reduces load on the backends. Never serve one user’s personal data to another user.
* **Security.** Backends are not exposed directly to the internet. The proxy can also limit request rates and filter bad requests.
* **Routing.** Send `/users` to the User Service and `/payments` to the Payment Service.

**Watch out:** single point of failure. Every request goes through the reverse proxy, so if it goes down, the whole site is unreachable. The usual fix is to run more than one proxy.

*Refer to the image here: `./diagrams/04-single-point-failure.svg`*

---

## Reverse proxy vs load balancer

People mix these up. A load balancer has one main job: spread incoming traffic across several servers. It decides which server gets each request using an algorithm, and the right choice depends on the situation:

* **Round robin.** Servers take turns. Good when the servers are similar and requests take about the same time.
* **Least connections.** The next request goes to the least busy server. Good when some requests are slow and others are quick.
* **Weighted.** Stronger servers get a bigger share. Good when servers have different capacity.
* **IP hash or sticky sessions.** The same user keeps reaching the same server. Good when a server keeps user data locally, but it can hide scaling problems.

A reverse proxy is the bigger idea. It also sits in front of the servers, and it can balance load, but it can also do TLS termination, caching, security and routing.

*Refer to the image here: `./diagrams/05-reverse-proxy-features.svg`*

NGINX, HAProxy and Envoy are reverse proxies that can also act as load balancers. In real systems you may not deploy something called a “reverse proxy” at all. A load balancer, API gateway, ingress controller or CDN may already be doing that work.

---

## Going a level deeper

The basics above are enough to explain the idea. The points below are what you run into when you actually use a reverse proxy.

### How does the backend know the real client IP?
The backend only talks to the proxy, so every request seems to come from the proxy’s IP. To fix this, the proxy adds the visitor’s IP in a header, usually `X-Forwarded-For`. It can also pass the original protocol (`X-Forwarded-Proto`) and the original host name. Without these headers, your logs and rate limits would treat all users as one.

### TLS termination: simple, but think about what’s behind it
The proxy decrypts HTTPS once, so backends don’t manage certificates. The trade-off is that traffic between the proxy and the backends may travel as plain HTTP. On a trusted private network that can be fine. For sensitive systems, encrypt that hop too.

### Layer 4 vs Layer 7

* **Layer 4** — Like a courier who reads only the address on the envelope. It forwards TCP or UDP connections without looking inside. Fast and simple, but it can’t route by URL.
* **Layer 7** — Like a courier who opens the letter first. It understands HTTP, so it can route by path, header or host name, for example `/payments` to the Payment Service.

### Caching safely
A cache is only safe for content that is the same for everyone, such as images or public pages. Personal pages (a user’s account or cart) must not be cached and shared, or one user could see another’s data.

---

## A tiny NGINX example

This is the smallest possible reverse proxy. NGINX accepts requests on port 80 and forwards every one to a backend server.

```nginx
server { 
    listen 80; 
    location / { 
        proxy_pass http://10.0.0.11:3000; 
    }
}
```

`listen 80` is the entry point visitors use. `location /` matches every URL path. `proxy_pass` sends the request on to the backend, and that one line is what makes NGINX a reverse proxy. The address is only an example. A real setup would add HTTPS, several backends, timeouts and logging.

---

## Common mix-ups

* **“The reverse proxy is a backend.”** No. It sits in front of the backends.
* **“A proxy makes me anonymous.”** No. It hides your IP from the website, but the proxy owner can see your traffic.
* **“The backend sees the real user IP automatically.”** No. It sees the proxy unless the proxy passes the IP in a header.
* **“Reverse proxy and load balancer are the same.”** Not exactly. Load balancing is one thing a reverse proxy can do.

---

## Quick recap

Both proxies sit in the middle. The forward proxy stands up for the user, the reverse proxy stands up for the servers. Ask yourself “who is this proxy protecting?” and you will always get the answer.

**One-line memory trick:** a forward proxy hides you from the website. A reverse proxy hides the servers from you.
