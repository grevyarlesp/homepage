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
- Users should be able to view blogs submitted by followed users, in form of a feed.
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

- Highly **available**
- Optimized for read
- Low lantecy (the blogs sent have to be fast)
- Don't have to be consistent (when a user posts a blog, we don't need to be immediately distributed to other users). 
- Have to be tolerance to our network changes and not lose our data.

## System APIs

- `POST blog(user_key, blog_data)`: returns the URL of new blog.
- `POST image(user_key, image_data)`: uploads an image, return URL of new image.
- `POST follows(user_key, user2_id)`: The user follows another user. Unfollow if the user is already following.
- `GET blog(blog_id)`: returns URL of blog
- `GET image(image_id)`: returns URL of image
- `GET follower(user_key)`: returns a list of other users that this user is following.

From this design, each of the feature expectations above can implemented.

This creates a bottleneck, if a user follow too many authors, the client has to do that many API calls to update his feed.

## High level design

<p align="center">
<img  src = "../img/design.svg" alt="High level design"/>
</p>

Explanation of the chart: 

- First, the client calls the load balancer. The load balancer divides up the workload between the web servers, it decides which server to handle each request. A level 4 load balancer should be sufficient.
- The server clusters: Receive and process requests from the users. 
- Cache: Cache for Database. The webserver also hits this first before hitting the DB (if not data is not cached).
- File storage stores our files: the markdown files and the image files.

For the requirements of low latency, it make senses to use a Content Delivery Network.

## Dive into each components

Let's take a dive into each component of our design.

### Load balancing

There are two types of load balancers: Hardware and software. 

- Hardware load balancers are network devices, they usually have many CPU cores, memory and very high throughput. 
- Software load balancers: installed on small devices, with load balancing softwares. Can also be scaled up horizontally.

There are several load balancing algorithms: 

- Round robin 
- Least connection 
- Least response time 

Round robin is a good choice in this scenario, as the actual bottleneck is not with the web servers nor our load balancers.

### Web servers

The web servers are built as a cluster. 

To ensure availability, we can divide add more web services, and use a service such as **Zookeeper**  for discovery and health check.

The load balancer can hit on **Zookeeper** to check which servers are avaiable, then send the requests to the available web servers.

The web servers employ **non-blocking IO** method to handle and process web requests. It involves a queue on a single thread to serve the requests that come first, hits the database then continues on with the next requests, and return to the clients when the data is returned from the database. This is better for our system horizontal scaling.

An alternative method is **blocking IO**, in which we uses a separate thread for every connection, and it doesn't do anything until the database return the results. This is much more expensive than using **non-blocking IO**, as it requires more **vertical scaling**.

The trade off for non-blocking IO is that it is more complex and we have less control over our requests, we won't know if our database died and not returning the requests at all.



### Database design

Define our tables: 

<p align="center">
<img  src = "../img/Schema.svg" alt="database"/>
</p>

With this simple database schema, we can fullfill all the above requirements. 

Users's personal information can be stored as a JSON string, as we don't have to search for those.

Our database can be designed as a cluster, with a 

**Replication** is a method of for availability, also a method for scaling. In our case, we use it ensure the availability, incase any database falls.

Two methods for replication:

- **Master-slave**: We have a single master and many slaves, which pull data from the master, serves as backup. Consistency is easy. In the event of master failure, a slave has to promoted to a master, and we have to restart our application.

- **Master-master**: Many masters share data with each other. In the event that a node goes down, the system can continue to operate. The trade-off that come with this is consistency. However, with many masters, our data can eventally be consistent.

<p align="center">
<img  src = "../img/db_design.svg" alt="cdn"/>
</p>

We need a load balancer for Master-master replication to speed up DB look up, a level 7 load balancer is approriate as it needs to know the content of the request.

We can use another cache to quickly look up commonly used searches.

### File storage

The markdown file stores references to our image ID, and the webserver can requests the files based on file ID (if they are stored in our system).

Content Delivery Network consists of many server globally, these are effectively caches, these are effectively caches.

<p align="center">
<img  src = "../img/design_with_cdn.svg" alt="cdn"/>
</p>


When web servers receive the `GET` requests, it tries to **pull** from the CDN first before hitting the actual file storage.

Based on the region, the web server would periodically **push** most requested data to the CDN.

The similar thing happens when the web server needs to request someting from the database.

## Bottlenecks and tradeoffs

No system is perfect, there has to some tradeoffs with our design.

A single load balancer can become a bottleneck and a single point of failure, for now one is enough. 




## External resources:

- The template from: https://leetcode.com/discuss/career/229177/My-System-Design-Template
- https://github.com/donnemartin/system-design-primer
- This [video](https://youtu.be/bUHFg8CZFws), very informative, explains the operation of each of the components well. 
- **Designing Data-Intensive Applications** book by Martin Kleppmann, informative on the design of each of the components.