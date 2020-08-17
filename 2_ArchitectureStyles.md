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
