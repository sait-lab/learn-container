## Docker Networking



Watch this YouTube video: [Docker networking is CRAZY!! (you NEED to learn it) (youtube.com)](https://www.youtube.com/watch?v=bKFMS5C4CG0)



---



### Networking Overview

Excerpt from [Networking overview | Docker Docs](https://docs.docker.com/network/)

>Container networking refers to the ability for containers to connect to and communicate with each other, or to non-Docker workloads.
>
>Containers have networking enabled by default, and they can make outgoing connections. A container has no information about what kind of network it's attached to, or whether their peers are also Docker workloads or not. A container only sees a network interface with an IP address, a gateway, a routing table, DNS services, and other networking details. That is, unless the container uses the `none` network driver.



---



### Default Bridge Network

Run the demos on [Networking with standalone containers | Docker Docs](https://docs.docker.com/network/network-tutorial-standalone/)



---



### Host Network

Run the demos on [Networking using the host network | Docker Docs](https://docs.docker.com/network/network-tutorial-host/)

