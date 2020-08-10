---
layout: post
title: Moving My Blogs to Netlify
---
For the last 3 years I've been hosting this and a few other blogs on a [Linode](https://www.linode.com) VPS with Nginx.
My sites pages are static contents generated using [Jekyll](https://jekyllrb.com) so I needed only http service to run the sites, so running a full VPS is simultaneously an overkill and not as robust as I like.

I experimented with cloud storage solutions (e.g. AWS S3) which works quite well except that I needed to also run CloudFront to get HTTPS.
Setting it up was a bit of a hustle so I never got around to it.

Fast forward to July 2020, I'm exploring [Netlify](https://www.netlify.com) for [Jamstack](https://jamstack.org) deployment.
Seeing that static web hosting with free HTTPS is specifically supported, I decided to point Netlify to my blog repository on Github to see what happens. 

It started building the site, guided me to set up the DNS, and set up the [LetsEncrypt](https://letsencrypt.org) certificates automatically for HTTPS.
After verifying that the site is indeed working as expected, I moved all my remaining blogs to Netlify and retired the server on Linode.

Netlify is not the only service for HTTPS enabled static sites with CDN.
[Firebase Hosting](https://firebase.google.com/products/hosting/) seems to be another credible alternative though I have not tried it.

The Netlify experience was so seamless that I am missing very little for my specific use case.
Definitely recommended if you need a similar service.
