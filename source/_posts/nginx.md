---
title: Nginx
date: 2025-03-12 14:41:36
tags:
    - system design
    - proxy
categories:
    - Nginx
---

# What is Nginx?
Nginx is an open-source Web Server and Reverse Proxy Server. It is famous for its high performance and high scalability. It has become the most popular Web Server besides Apache. It is responsible for acting as a bridge between the Client and the Server and can perform multiple proxy functions such as HTTP/HTTPS request, Load Balancing and Cache.

This chapter is more inclined to explore the internal architecture of Nginx. From the perspective of Nginx architecture, we can understand the types of Nginx workers, how Nginx receives and handles the problem of simultaneous C10K, and so on.

<!-- more -->

# The Nginx Process Model
Nginx has two part, master process and worker process.

![Nginx Process](https://ithelp.ithome.com.tw/upload/images/20250312/20124767fs4jW8fYht.png)

## Master Process
Master process is responsible for coordinating and managing the worker processes. It listen requests and manage the configuration and overall operation of the worker process.
- Read and validate configuration when Nginx starts
- Operate the socket that receives the request (add, bind, close)
- Start, terminate and maintain the worker processes and manage the number of worker processes
- Support and implement update Nginx configuration dynamically

## Worker Process
Compare with the Master process, the worker process has a relatively simple tasks. Its main purpose is to process the request from the client, provide feature of reverse proxy and filter responses from the Server. In short, all the feature of Nginx actual application is completed through the Worker Processes.
1. After Nginx receives the request, The Master Process assigns this request to one of the Worker Process
2. The Worker process checks if the request hits the cache by The Cache key
3. If the cache is hit, the Worker Process will read the cache content instead of sending the request to the backend server (Web Server).
4. The Worker Process will return cache content to the client.
    ```
    HTTP/1.1 200 OK
    X-Cache: HIT
    Content-Type: text/html
    ```
5. If it doesn't hit, the request is forwarded to the subsequent server

## Cache Worker Processes
Nginx's Cache worker can temporarily store previously processed responses in memory or disks. When a request accesses the response, Nginx can direct obtain the previously store content without access backend server again, which can effectively improve access efficiency.
### Cache Manager Process
The Cache Manager Process is a system which used to manage caches. It periodically checks cached contents that which content are expired or no longer used, and delete them to avoid the cache from becoming too large.

### Cache Loader Process
The Cache Loader Process is used to store responses from Web Server or other sources into Cache. After Nginx starts, Cache Loader Process will scan the cache directory and load the contents into memory. After complecation, the Process will exit. 

---

# How does the Nginx work?
## Traditional server architecture (Apache’s prefork)
In the traditional server model, the server will create a separate thread or process for each request to handle it. This dedicated thread or process will block other requests during the request processing until the request is completed.

Although this allow a thread or process focus on single request, this behavior is sometimes very inefficient. Create a thread or process for single request requires a lot of resources (allocating memory, creating a new execution environment...). Creating those resources will consume CPU time and it will also cause race condition or poor performance when switching between different threads or processes.

## How about Nginx?
Nginx doesn't create new process for each request, instead of create specified number of workers (usually the number of CPUs) according to the configuration when Nginx starts
```
worker_processes auto;
```
This ensure that each workers uses a core of CPU separately, and there will be no problem of insufficient source. All the workers will be ready to receive the requests and Nginx will assigns requests to workers in different ways through configuration.

1. **accept_mutex**: If accept mutex is enabled, the workers will `take turns receiving requests`. It helps reduce worker's competition for requests and ensure load balancing among workers.
2. **reuseport**: Can set reuseport to make each worker listens to a different port to avoid multi workers waiting for requests from the same port. If the traffic on a specific port is very large, can set multi workers to listen to this port together and distribute requests to those workers randomly or in a round-robin manner.
![reuseport](https://ithelp.ithome.com.tw/upload/images/20250312/20124767hTjbx8i5Ow.png)
3. **worker_connections**: Nginx will adjust who to receive the request based on  the load of each worker. If a worker handles too many requests, the next request won't send to this worker, will be sent to a worker with a relatively low load for processing, but this distribution will `not strictly guarantee load balancing`.

Although Nginx is not like traditional server which create a new thread or process for a request, when will the worker send the response back to the client after receiving the request?
This is another design of Nginx that can achieve this is introduced, which is event-driven. 

Nginx continuously receives requests in an asynchronous manner and sends the responses back to the client in an event-driven manner, which is somewhat similar to the event loop concept of JavaScript for handling asynchronous tasks.

## Event-Driven Programming in Nginx
Event-Driven is a software design pattern, this design pattern allow applications to handles events in an asynchronous manner, unlike the traditional blocking mode, need to wait until previous task to complete before executing the next one.

Nginx uses an event-driven architecture, which allows it to handle a large number of concurrent requests (high concurrency requests) at the same time without affecting other requests due to waiting for a certain request to complete. Nginx will repeatedly check if there are new events or responses returned. If there are, it will perform corresponding processing according to the Configuration settings (prioritize request processing, priority response return...).
![Event-Driven](https://ithelp.ithome.com.tw/upload/images/20250312/20124767nAzDZA6ifS.png)

### How nginx prcess requerst by Event-Driven?
Nginx processes requests coming into the worker through Asynchronous I/O and Event Loop. The main process is as follows:
1. **Master Worker Process receives request**
    - When the client sends a request, the Nginx Master Worker Process receives these requests in a non-blocking manner and puts them in the event-queue.
2. **The worker process will not wait for this request to complete.**
    - Unlike the traditional blocking mode, the worker process will not be stuck waiting for the request to be completed, but will continue to receive new requests. The advantage of this is that even if it takes a long time to process a request, the worker process will not be stuck and can continue to receive other requests.
3. **Event loop continuously checks the status of the request**
    - Worker manages all received requests through epoll (Linux) or kqueue (BSD/macOS)
    -  If the server returns a response, Nginx will perform corresponding operations based on the configuration and request type (return static files to the client, forward the request again...)
4. **Send back large amounts of data in chunks**
    - If the Worker receives a large amount of response data from the Server, it will use chunked transfer encoding to ensure that the returned data is transmitted step by step instead of returning a large packet of data to the client at once, increasing the reliability of the client receiving data.

---

# Reference
- [SoulaimaneyhExplaining Nginx Internal Architecture](https://medium.com/@soulaimaneyh/nginx-internal-architecture-b94b013bc365)
- [NGINX Community BlogInside NGINX: How We Designed for Performance & Scale](https://blog.nginx.org/blog/inside-nginx-how-we-designed-for-performance-scale)
- [NGINX Community BlogPerformance Tuning – Tips & Tricks](https://blog.nginx.org/blog/performance-tuning-tips-tricks)
- [The Architecture of Open Source Applications (Volume 2)nginx](https://aosabook.org/en/v2/nginx.html#fig.nginx.arch)

