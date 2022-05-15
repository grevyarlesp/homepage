---
title: "Designing a scalable blog platform"
date: 2022-05-10T13:08:18+07:00
draft: false
---

A classic system design interview question is to design a blog platform. For this particular question, we design a system that can handle to 10000 requests per second.

## Feature expectations:

Since the question is very open-ended with no clear requirements, I am making up some feature requirements:

- Users can register and log in into their accounts
- Users should be able to write their blogs in Markdown form (provides a markdown editor in client side)
- Users can follow other users.
- Users should be able to view blogs submitted by other users, in form of a feed.
- Blogs can contain images, but no videos.

## Estimations:

These are not accurate numbers, I looked them up from various sources to make our hypothetical design easier to work with:

- Each requests a blog, or send a blog to the server.
- Each blog has 10000 characters (2 bytes for each, 20000 bytes), so each blog is 20KB.
- Each blog may includes image, let's limit each to 2048 pixels (as Blogger does), so each image's size is around $2048 * 3 = 6144$. An average blog has 10 images (my assumption), so the total image size is around 12KB. Total size of an average blog post 32KB.
- **Traffic**: We have to serve 10000 requests per second, that puts it at $32 * 10000 = 320000$ KB, or 320MB of throughput per second. Compression is not considered here.
- **Storage:** Each of our users submits 2-3 blogs every day, let's put it at 20000-30000 blogs, so each month we have roughly 30GB more data. Or $30 * 365 \approx 11$ TB per year.

## Design goals

We want our system to be:

- Highly available 
- Optimized for read
- Low lantecy (the blogs sent have to be fast)
- Don't have to be very consistent (when a user posts a blog, we don't need to be immediately distributed to other users feed)

## System APIs

- `POST blog(user_key, blog_data)`: returns the URL of new blog.
- `POST image(user_key, image_data)`: uploads an image, return URL of new image.
- `GET blog(blog_id)`: returns URL of blog
- `GET image(image_id)`: returns URL of image

## High level design

![](./img/design.svg)

We can sketch out our high level design: 

- First, the client calls the load balancer. The load balancer divides up the workload between the web servers, it decides which server to handle each request. A level 4 load balancer should be sufficient.
- The server clusters:
- File storage stores our files: the markdown files and the image files.

## Deep dive

Let's take a deep dive into each component of our design.

### Database schemas





## References:

- The template from: https://leetcode.com/discuss/career/229177/My-System-Design-Template
- https://github.com/donnemartin/system-design-primer