---
title: "This week in tech news | 8th October 2022"
seoTitle: "This week in tech news | 8th October 2022"
datePublished: Sun Oct 09 2022 15:41:06 GMT+0000 (Coordinated Universal Time)
cuid: cl91iiuwh001709mh9nl01uxj
slug: this-week-in-tech-news-8th-october-2022
cover: https://cdn.hashnode.com/res/hashnode/image/unsplash/jLwVAUtLOAQ/upload/v1665330036940/xtIwLzsAW.jpeg
tags: cloudflare, postgresql, wasm, supabase

---



# Cloudflare creates own Reverse Proxy replacing Nginx

Cloudflare has created its own reverse proxy, replacing Nginx with a in-house built proxy called Pingora built using Rust. This was built due to poor connection pool resource allocation in Nginx at Cloudflare's scale which resulted to slower TTFB(Time to first byte). After the migration, connection reuse ratio improved; as a an example Cloudflare quoted "increased the connection reuse ratio from 87.1% to 99.92%, which reduced new connections to their origins by 160x. To present the number more intuitively, by switching to Pingora, we save our customers and users 434 years of handshake time every day"
[Cloudflare blog:](https://blog.cloudflare.com/how-we-built-pingora-the-proxy-that-connects-cloudflare-to-the-internet/)


![933CF66E-C0FD-410A-BCEA-10291CE251F4.jpeg](https://cdn.hashnode.com/res/hashnode/image/upload/v1665329946255/ae72zckjR.jpeg align="left")


# Run Postgres in the browser
Postgres-wasm built by Supabase and Snaplet to bring Postgres to the browser with the help of WASM. 
postgres-wasm runs a small distro called buildroot to create a 12MB snapshot for the Postgres server to be run on the browser.
Other features include creating a exporting to IndexedDB(browser) or to disk, connectivity between browser and external source.

### [Play around here: ](https://wasm.supabase.com/)

![7CA3E145-09C6-4608-89D6-487F04A313BA.jpeg](https://cdn.hashnode.com/res/hashnode/image/upload/v1665329958122/rRmNb0PYV.jpeg align="left")
### [Supabase blog:](https://supabase.com/blog/postgres-wasm)


![3229F94E-184D-444F-926B-21D99D51B1C6.jpeg](https://cdn.hashnode.com/res/hashnode/image/upload/v1665329973450/mo6ybO4Mu.jpeg align="left")
