---
title: "Server-Sent Events with Nuxt.js (Vue.js) and Nginx proxy"
date: "2017-11-30"
draft: false
tags: [SSE, Nuxt.js, Vue.js, node, Nginx, event stream]
featured: true
---

Suppose you have a Nuxt.js (Vue.js) application which calls a backend and which is behind a Nginx reverse proxy. If your backend stream Server-Sent Events (SSE) using `text/event-stream` content-type, you may have these errors: `504 (Gateway Time-out)` or `ERR_INCOMPLETE_CHUNKED_ENCODING` (in Chrome).

If you want to make SSE works with your Nuxt.js application which is behind a Nginx reverse proxy, you have to do 2 things :

### Enable long running connections in Nginx

Add these lines in your Nginx configuration:
```
proxy_http_version 1.1;
proxy_set_header Connection "";
```

And don't forget to add the `X-Accel-Buffering: no;` header in the stream response sent by your backend. Full explanation in this answer: [https://serverfault.com/a/801629](https://serverfault.com/a/801629)

### Disable gzip compression in Nuxt

Because compression does not work well with event streams. Add these lines in your `nuxt.config.js` configuration file:
```
render: {
  gzip: false
}
```

More configuration available here: [https://nuxtjs.org/api/configuration-render](https://nuxtjs.org/api/configuration-render)

_That's all folks!_
