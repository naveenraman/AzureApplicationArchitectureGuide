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

