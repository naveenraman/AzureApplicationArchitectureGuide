# Architecture styles
- A family of architectures that share certain characteristics.
- These don't require that use of particular technologies, but some technologies are well-suited for certain architectures.
- Each style includes the following details:
    - a description
    - logical diagram
    - recommendations for when to choose this style
    - benefits
    - challenges
    - best practices
    - a recommended deployment using relevant Azure services
    
## N-tier
- Dependencies are managed by dividing the application into logical layers and physical tiers such as presentation, business logic, and data access.

  ![](/images/n_tier.png)

- A layer can only call into layers that sit below it.
- Horizontal layering a liability, it can be hard to introduce changes in one part of the application without touching the rest of the application. Frequent updates are challenging, limiting how quickly new features can be added.
- Tiers are physically separated, running on separate machines. A tier can call to another tier directly, or use asynchronous messaging (message queue).
- Physically separating the tiers improves scalability and resiliency, but also adds latency from the additional network communication.
- An N-tier application can have a closed layer architecture or an open layer architecture:
    - In a closed layer architecture, a layer can only call the next layer immediately down.
    - In an open layer architecture, a layer can call any of the layers below it.
  A closed layer architecture limits the dependencies between layers.

#### When to use this architecture
- N-tier is best fit for migrating existing applications that already use a layered architecture.
- N-tier is most often seen in **infrastructure as a service (IaaS)** solutions, or application that use a mix of Iaas and managed services.
- N-tier architecture can be considered for following applications:
    - Simple web applications
    - Migrating an on-premises application to Azure with minimal refactoring
    - Unified development of on-premises and cloud applications.

#### Benefits
- Portability between cloud and on-premises, and between cloud platforms.
- Less learning curve for most developers.
- Natural evolution from the traditional application model.
- Open to heterogeneous environment (Windows/Linux)

#### Challenges
- It's easy to end up with a middle tier that just does CRUD operations on the database, adding extra latency without doing any useful work.
- Monolithic design prevents independent deployment of features.
- Managing an IaaS application is more work than an application that uses only managed services.
- It can be difficult to manage network security in a large system.

#### Best practices
- Use autoscaling to handle changes in load.
- Use asynchronous messaging to decouple tiers.
- Cache semistatic data.
- Configure the database tier for high availability, using a solution such as SQL Server Always On availability groups.
- Place a web application firewall (WAF) between the front end and the Internet.
- Place each tier in its own subnet, and use subnets as a security boundary.
- Restrict access to the data tier, by allowing requests only from the middle tier(s).

#### Additional considerations
- N-tier architectures are not restricted to three tiers. For more complex applications, it is common to have more tiers. In that case, consider using layer-7 routing to route requests to a particular tier.
- Tiers are the boundary of scalability, reliability, and security. Consider having separate tiers for services with different requirements in those areas.
- Use virtual machine scale sets for autoscaling.
- Look for places in the architecture where you can use a managed service without significant refactoring like caching, messaging, storage and databases.
- For higher security, place a network DMZ in front of the application. The DMZ includes network virtual appliances (NVAs) that implement security functionality such as firewalls and packet inspection.
- For high availability, place two or more network virtual appliances (NVAs) in an availability set, with an external load balancer to distribute Internet requests across the instances.
- Do not allow direct RDP or SSH access to VMs that are running application code. Instead, operators should log into a jumpbox, also called a bastion host. This is a VM on the network that administrators use to connect to the other VMs. The jumpbox has a network security group that allows RDP or SSH only from approved public IP addresses.
- You can extend the Azure virtual network to your on-premises network using a site-to-site virtual private network (VPN) or Azure ExpressRoute.
- If your organization uses Active Directory to manage identity, you may want to extend your Active Directory environment to the Azure VNet.
- If you need higher availability than the Azure SLA for VMs provides, replicate the application across two regions and use Azure Traffic Manager for failover.

#### Recommendation for N-tier architecture on virtual machines
![](/images/n_tier_vms.PNG)

- Each tier consists of two or more VMs, placed in an availability set or virtual machine scale set.
- Multiple VMs provide resiliency in case one VM fails.
- Load balancers are used to distribute requests across the VMs in a tier.
- A tier can be scaled horizontally by adding more VMs to the pool.
- Each tier is also placed inside its own subnet, meaning their internal IP addresses fall within the same address range and making it easy to apply network security group (NSG) rules and route tables to individual tiers.
- The web and business tiers are stateless.Any VM can handle any request for that tier.
- The data tier should consist of a replicated database. For Windows, we recommend SQL Server, using Always On availability groups for high availability. For Linux, choose a database that supports replication, such as Apache Cassandra.
- Network security groups restrict access to each tier.(For example, the database tier only allows access from the business tier.)

## Web-Queue-Worker
- The core components of this architecture are a web front end that serves client requests, and a worker that performs resource-intensive tasks, long-running workflows, or batch jobs. The web front end communicates with the worker through a message queue.

![](/images/web_queue_worker.png)

- Other components that are commonly incorporated into this architecture include:
    - One or more databases.
    - A cache to store values from the database for quick reads.
    - CDN to serve static content
    - Remote services, such as email or SMS service.
    - Identity provider for authentication.
- The web and worker are both stateless. Session state can be stored in a distributed cache. 
- Any long-running work is done asynchronously by the worker. The worker can be triggered by messages on the queue, or run on a schedule for batch processing. The worker is an optional component. If there are no long-running operations, the worker can be omitted.
- The front end might consist of a web API.
- On the client side, the web API can be consumed by a single-page application that makes AJAX calls, or by a native client application.

#### When to use this architecture
- The Web-Queue-Worker architecture is typically implemented using managed compute services, either Azure App Service or Azure Cloud Services.
- Web-Queue-Worker architecture can be considered for following applications:
    - Applications with a relatively simple domain.
    - Applications with some long-running workflows or batch operations.
    - When you want to use managed services, rather than **infrastructure as a service (IaaS)**.

#### Benefits
- Relatively simple architecture that is easy to understand.
- Easy to deploy and manage.
- Clear separation of concerns.
- The front end is decoupled from the worker using asynchronous messaging.
- The front end and the worker can be scaled independently.

#### Challenges
- Without careful design, the front end and the worker can become large, monolithic components that are difficult to maintain and update.
- There may be hidden dependencies, if the front end and worker share data schemas or code modules.

#### Best practices
- Expose a well-designed API to the client.
- Autoscale to handle changes in load.
- Cache semi-static data.
- Use a CDN to host static content.
- Use polyglot persistence when appropriate.
- Partition data to improve scalability, reduce contention, and optimize performance.

#### Additional considerations
- Not every transaction has to go through the queue and worker to storage. The web front end can perform simple read/write operations directly. Workers are designed for resource-intensive tasks or long-running workflows. In some cases, you might not need a worker at all.
- Use the built-in autoscale feature of App Service to scale out the number of VM instances. If the load on the application follows predictable patterns, use schedule-based autoscale. If the load is unpredictable, use metrics-based autoscaling rules.
- Consider putting the web app and the function app into separate App Service plans. That way, they can be scaled independently.
- Use separate App Service plans for production and testing. Otherwise, if you use the same plan for production and testing, it means your tests are running on your production VMs.
- Use deployment slots to manage deployments. This lets you to deploy an updated version to a staging slot, then swap over to the new version. It also lets you swap back to the previous version, if there was a problem with the update.

#### Recommendation for Web-Queue-Worker on Azure App Service
![](/images/web_queue_worker_azure_app_service.png)

- The front end is implemented as an Azure App Service web app.
- The worker is implemented as an Azure Functions app.
- The web app and the function app are both associated with an App Service plan that provides the VM instances.
- We can use either Azure Service Bus or Azure Storage queues for the message queue.
- Azure Cache for Redis stores session state and other data that needs low latency access.
- Azure Content Delivery Network (CDN) is used to cache static content such as images, CSS, or HTML.
- For storage, choose the storage technologies that best fit the needs of the application.
- You might use multiple storage technologies (polyglot persistence).

## Microservices
- A microservices architecture consists of a collection of small, autonomous services. Each service is self-contained and should implement a single business capability.

![](/images/microservices.png)

#### What are microservices?
- Microservices are small, independent, and loosely coupled. A single small team of developers can write and maintain a service.
- Each service is a separate codebase, which can be managed by a small development team.
- Services can be deployed independently. A team can update an existing service without rebuilding and redeploying the entire application.
- Services are responsible for persisting their own data or external state. This differs from the traditional model, where a separate data layer handles data persistence.
- Services communicate with each other by using well-defined APIs. Internal implementation details of each service are hidden from other services.
- Services don't need to share the same technology stack, libraries, or frameworks.

*Management/orchestration*
- This component is responsible for placing services on nodes, identifying failures, rebalancing services across nodes, and so forth.
- This component is Kubernetes.

*API Gateway*
- The API gateway is the entry point for clients. Instead of calling services directly, clients call the API gateway, which forwards the call to the appropriate services on the back end.
- Advantages:
    - It decouples clients from services. Services can be versioned or refactored without needing to update all of the clients.
    - Services can use messaging protocols that are not web friendly, such as AMQP.
    - The API Gateway can perform other cross-cutting functions such as authentication, logging, SSL termination, and load balancing.
    
#### Benefits

- *Agility*
    - Microservices are deployed independently, it's easier to manage bug fixes and feature releases.
    - We can update a service without redeploying the entire application, and roll back an update if something goes wrong.

- *Small, focused teams*
    - A microservice should be small enough that a single feature team can build, test, and deploy it.
    - Small team sizes promote greater agility.
    
- *Small code base*
    - By not sharing code or data stores, a microservices architecture minimizes dependencies, and that makes it easier to add new features.
    
- *Mix of technologies*
    - Teams can pick the technology that best fits their service, using a mix of technology stacks as appropriate.

- *Fault isolation*
    -  If an individual microservice becomes unavailable, it won't disrupt the entire application, as long as any upstream microservices are designed to handle faults correctly e.g. implementing circuit breaking.

- *Scalability*
    - Services can be scaled independently, letting you scale out subsystems that require more resources, without scaling out the entire application.
    - Using an orchestrator such as Kubernetes or Service Fabric, you can pack a higher density of services onto a single host, which allows for more efficient utilization of resources.

- *Data isolation*
    - It is much easier to perform schema updates, because only a single microservice is affected.

#### Challenges

- *Complexity*
    - A microservices application has more moving parts than the equivalent monolithic application. 
    - Each service is simpler, but the entire system as a whole is more complex.

- *Development and testing*
    - Writing a small service that relies on other dependent services requires a different approach than a writing a traditional monolithic or layered application.
    - Existing tools are not always designed to work with service dependencies. 
    - Refactoring across service boundaries can be difficult. 
    - It is also challenging to test service dependencies, especially when the application is evolving quickly.

- *Lack of governance*
    - The decentralized approach to building microservices has advantages, but it can also lead to problems. 
    - You may end up with so many different languages and frameworks that the application becomes hard to maintain. 
    - It may be useful to put some project-wide standards in place, without overly restricting teams' flexibility. 
    - This especially applies to cross-cutting functionality such as logging.

- *Network congestion and latency* 
    - The use of many small, granular services can result in more interservice communication. 
    - Also, if the chain of service dependencies gets too long (service A calls B, which calls C...), the additional latency can become a problem. 
    - You will need to design APIs carefully. 
    - Avoid overly chatty APIs, think about serialization formats, and look for places to use asynchronous communication patterns.

- *Data integrity*
    - With each microservice responsible for its own data persistence. As a result, data consistency can be a challenge. 
    - Embrace eventual consistency where possible.

- *Management*
    - To be successful with microservices requires a mature DevOps culture. 
    - Correlated logging across services can be challenging. Typically, logging must correlate multiple service calls for a single user operation.

- *Versioning*
    - Updates to a service must not break services that depend on it. 
    - Multiple services could be updated at any given time, so without careful design, you might have problems with backward or forward compatibility.

- *Skillset*
    - Microservices are highly distributed systems. 
    - Carefully evaluate whether the team has the skills and experience to be successful.

#### Best practices
- Model services around the business domain.
- Decentralize everything. Individual teams are responsible for designing and building services. Avoid sharing code or data schemas.
- Data storage should be private to the service that owns the data. Use the best storage for each service and data type.
- Services communicate through well-designed APIs. Avoid leaking implementation details. APIs should model the domain, not the internal implementation of the service.
- Avoid coupling between services. Causes of coupling include shared database schemas and rigid communication protocols.
- Offload cross-cutting concerns, such as authentication and SSL termination, to the gateway.
- Keep domain knowledge out of the gateway. The gateway should handle and route client requests without any knowledge of the business rules or domain logic. Otherwise, the gateway becomes a dependency and can cause coupling between services.
- Services should have loose coupling and high functional cohesion. Functions that are likely to change together should be packaged and deployed together. If they reside in separate services, those services end up being tightly coupled, because a change in one service will require updating the other service. Overly chatty communication between two services may be a symptom of tight coupling and low cohesion.
- Isolate failures. Use resiliency strategies to prevent failures within a service from cascading.

