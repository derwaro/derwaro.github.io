---
title: "Retrieve Ngrok Url Dynamically"
date: 2023-08-21T23:10:38-06:00
lastmod: 
draft: false
tags: ["ngrok", "python", "api"]
categories: ["tech"]
---

Ngrok does not offer fixed url/domains in their free plan. BUT ngrok offers us an api :)

I will only get into one aspect of their api to leverage it and obtain a paid feature for free through an extra request and some line of code. 

## The Problem
### General
As I already mentioned, having a static url/domain in ngrok is a paid feature and starts at the moment of writing this post at 8 Dollars/month (See [ngrok-pricing](https://ngrok.com/pricing) for more information).
The only feature I'm missing in the free plan is having a static url/domain, so that I don't have to change that variable every time I start checking stuff in my apps. I want to start ngrok and go on.

### More Specific Example
I encountered this little problem when having an endpoint in fastAPI that places a request at an external API. This external API sends me some feedback, but only to a publicly accessible url (sorry, localhost). When I'm running my app on my server I have no problem, by nature the app is accessible and the external API can send me whatever it wants to send me. However while developing it is nice to do some manually tests, trial/error,... therefore I opted for ngrok and their tunneling feature, which provisions a random url on each startup and tunnels it to the beloved localhost. To reduce costs, I'm using the free tier of ngrok, which on each startup of ngrok (e.g. `ngrok http 8000`) assigns me a random long string url. This url has than to be copied into my endpoint. This is pretty repetitive, so I ventured out for an automated solution.

## The Fix
### General
ngrok serves a really nice dashboard to inspect the requests and responses it receives. One could scrape the random/forwarding/public url from there. But to be honest, I don't have any experience scraping. But I have some experience querying APIs. And it just so happens that ngrok also serves a local api with it's web interface: (Source from ngrok documentation)[https://ngrok.com/docs/secure-tunnels/ngrok-agent/reference/api/]. There is even one (endpoint)[https://ngrok.com/docs/secure-tunnels/ngrok-agent/reference/api/#request] that serves all the information one would normally see in the terminal where ngrok was launched. So to retrieve the public url from ngrok dynamically all that has to be done is query this endpoint and extract the right parameter.

### Code example for python
This example first checks in what environment it is run and from there either sets the `status_callback_url` as the app's public url if it is run in `production` or extracts the ngrok public url from it's api as described.
```python
# set url for callbacks on call events, based on current environment from config.env
if config.ENVIRONMENT == "production":
    status_callback_url = "wwww.yourdomain.com/callback"
else:
    # have ngrok started!
    # leverages ngrok api to retrieve the public url dynamically
    async with httpx.AsyncClient() as as_client:
        url = "http://localhost:4040/api/tunnels/"
        response = await as_client.get(url)
        public_url = response.json()["tunnels"][0]["public_url"]
    status_callback_url = f"{public_url}" + f"/callback"
```
The only prerequisite is to start ngrok before executing this bit of code (`ngrok http {port of your development server}`)

Source: https://ngrok.com/docs/secure-tunnels/ngrok-agent/reference/api/#request
Inspiration: https://stackoverflow.com/questions/63492743/how-to-pipe-the-result-of-ngrok-http-8000-to-another-file