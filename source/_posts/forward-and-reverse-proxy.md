---
title: Forawrd Proxy and Reverse Proxy in system design
date: 2025-02-18 15:33:22
tags:
    - system design
    - proxy
categories:
    - system design
---

# Forward Proxy
![Forward Proxy](https://ithelp.ithome.com.tw/upload/images/20250218/20124767RGyo6xkbfM.png)


## What is Forward Proxy?
A Forward Proxy acts as a bridge between the client side and the server side. All requests from the client side sent to proxy server which then forwards them to the server side.
<!-- more -->
This setup allows the proxy to manage traffic, enhance security and keep client requests anonymous.

## What benefits does Forward Proxy provide?
Imagine a classroom where the teacher represents the server side, the classmates represent client requests, and the class leader acts as the forward proxy server.
Instead of each classmates asking the teacher questions directly, they relay questions to the class leader.The class leader forward all qeustions to the teacher. The teacher answers questions back to the class leader, whe then response them to classmates.

1. **Anonymity and Privacy**: Since all requests come from the forward proxy server, the server side can't know which client made the request. This system helps maintain client anonymous.

    > The teacher only know that the questions come from the class leader, but can't know which classmate originally asked them.

2. **Bypassing access restrictions**: Although the server side has access restrictions in place, client can still send requests to the proxy server. Since the server can't know which client made this request, only know the reqeust come from proxy server, this setup allows clients to bypass access restrictions.

    > Even if the teacher is only supposed to answer questions from their own students, a student who from another class can still ask question through the class leader. The class leader then forward this question to the teacher. 
    Since Teacher can't know who originally asked this quesiton (even asked by other class's student) and the teacher only know the question come from class leader, students from other classes can also get theri questions answered.

3. **Filter content and cache**: The proxy server can cache responses after accessing the server. This allows that proxy to return cached responses directly when a client requests the same resource again, without needing to access the server repeatedly.
This system helps reduces bandwidth consumption and improves efficiency.

    > If many students want to ask the same question without a class leader, the teacher would have to answer the same question repeatedly.
    Instead, the class leader can first receive this question, forward it to the teacher, and get answer. Once the class leader knows the answer it can respond directly to classmates without needing to ask the teacher again.

4. **Monitor and filter traffic to improve server side security**: Since all requests must go through the proxy server, it can monitor and record the source of requests and enforce limitations if necessary. This system helps enhance server secuirty by filting traffic and preventing unauthorized access

    > Since all questions from classmates must go through the class leader, the class leader can filter out meaningful questions and eliminate malicious ones. This process helps protect the teacher by preventing unnecessary or harmful questions from reaching them

5. **protocol translation**: Even if the server uses the HTTPS protocol, not all clients support HTTPS. Client who only support HTTP would fail to access the server directly. However, they can route their reqeusts through the proxy server, which translates HTTP into HTTPS. SInce the server only know requests coming from the proxy server and recognizes the reqeusts as HTTPS, the server can responds it without knowing that the original reqeuet was. 

    > In a bilingual school, the teacher is from the USA and only speaks English, while some students only speak Chinese. If these students need to ask questions, they must go through the class leader who can speak both Enghish and Chinese. The class leader translates their questions into English before asking the teacher and then translates the teacher's response back into Chinese for the students.

## Implementation forward proxy server
I created a sample forward proxy server using Node.js. This project was developed to verify and implement the benefits of a forward proxy server, such as request routing, caching, security filtering.

[forward_proxy_project](https://github.com/Fandix/forward_proxy_project)

---
# Reverse Proxy
![reverse proxy](https://ithelp.ithome.com.tw/upload/images/20250219/20124767cQPxjA1Wfq.png)
A reverse proxy also acts as a bridge between the client side and the server side. All requests from the client side sent to the proxy server which then forwards them to the appropriate server.
Unlike the forward proxy where the server can't identify the original cient, in a reverse proxy setup, the client doesn't know which specific server handels the request.

## BTW
As the website grows in popularity, a single request can quickly multiply into hundreds of thousands of reqeusts. One server alone may not be sufficient to handle this load, so upgrades are necessary.
There ary two type of upgrades, vertical expansion and horizontal expansion.
- **vertical expansion**: vertical expansion is upgrading a single server to make it more powerful, enabling it to handle large reqeusts more quickly and efficiently.
- **horizontal expansion**: Horizontal expansion is distributing the workload across many servers. Each server is either responsibility for a specific task or shares the large volume of request at same time. This approach enhances scalability and reliability by balancing the load among many servers.

Although both approaches can resolve server blocking issues caused by large or requests, vertical expansion is more costly. Therefore, horizontal expansion is generally preferred for upgrading servers to enhance their capacity and performace

## What benefits does Reverse Proxy provide?
imagine a bank representing the server side, the customer service center acting as the reverse proxy and clients with account-related questions as internet requests.
1. **Load Balancing**: When a large volume of requests access the reverse proxy, it can use specialized techniques to distribute the requests across multiple servers. This system helps prevent any single server blocking cause faced large volume of requests, ensuring better load balancing and avoiding blocking during high-traffic periods
    > When a large number of clients call with account-related questions, the customer service center can distribute the calls across multiple department. For example, calls about savings accounts can be routed to the saving-account department, while investment-related inquiries are directed to the investment department. This approach ensures that each request is handled by the most relevant department, optimizing efficiency  and preventing any single department overload.
2. **Caching**: Just like the forward proxy can cache responses after accessing the server. This allows that proxy to return cached responses directly when a client requests the same resource again, without needing to access the server repeatedly. This system helps reduces bandwidth consumption and improves efficiency.
3. **SSL Encryption and Decryption**: SSL/TLS encryption and decryption can be centrally managed in the reverse proxy, allowing the the server to handle requests without the need for decryption. This approach reduces the server's processing load, enhancing its overall performance and efficiency.
    > Every clients needs to validate their identity before their issue can be handled by the related department. The identity validation can be centrally operated in the customer service center. This way, each department can focus solely on solving the problem without needing to handle identiry validation.
4. **Dynamic load balancing and failover**: The reverse proxy monitors the health of servers. If a server encounters an error and not able to handle requests, the reverse proxy redirects the reqeusts to other healthy servers. This system helps ensures continuous service availability and improves system reliability.
    > If all employees in a department go to lunch, the customer service center can redirects the calls or problems to another department for handling. This ensure that no issue is left unattended, even when one department is temporarily unavailable
5. **Hide the server side to increase system security**: Because all requests must go through reverse proxy, client can't know which server is handling the request. This systems helps conceal the server-side architecture, enhancing security by preventing direct access to the servers.
    > Since all clients must go through the customer service center, they can't know which specific department is handling the issue. This system helps conceal the internal structure of the bank, protecting sensitive operational details.

## Implementation reverse proxy server
I created a sample reverse proxy server using Node.js. This project was developed to verify and implement the benefits of a reverse proxy server, such as load balancing, caching, dynamic load balancing and failover.

[Reverse-Proxy-Project](https://github.com/Fandix/Reverse-Proxy-Project)

---

# Compare between forward and reverse proxy
![compare between forward and reverse](https://ithelp.ithome.com.tw/upload/images/20250219/20124767n9kwp9iwaB.png)
Both types of proxy servers act as a bridge between the client side and the server side, but they serve different purposes:

- **Forward Proxy**: It hides the client's information, enhancing privacy and security on the client side. Forward proxies are suitable for <u>maintaininga anonymity and bypassing access restrictions</u>.

- **Reverse Proxy**: It hides the server's information, enhancing scalability and reliability by distributing request across multiple servers. Reverse proxies are suitable for <u>managing large volumes of requests and balancing traffing traffic efficiently</u>.