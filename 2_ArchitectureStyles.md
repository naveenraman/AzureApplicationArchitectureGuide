# Table of Contents
[Table of Contents]: #table-of-contents
- [Architecture styles]
- [Different type of styles]
    - [N-tier architecture style]
        - [When to use this architecture]
        - [Benefits]
        - [Challenges]
        - [Best practices]
        - [Additional considerations]
        - [Recommendation for N-tier architecture on virtual machines]
    - [Web-Queue-Worker architecture style]
        - [When to use this architecture]
        - [Benefits]
        - [Challenges]
        - [Best practices]
        - [Additional considerations]
        - [Recommendation for Web-Queue-Worker on Azure App Service]
    - [Microservices architecture style]
        - [What are microservices?]
            - [Management/orchestration]
            - [API Gateway]
        - [Benefits]
            - [Agility]
            - [Small, focused teams]
            - [Small code base]
            - [Mix of technologies]
            - [Fault isolation]
            - [Scalability]
            - [Data isolation]
        - [Challenges]
            - [Complexity]
            - [Development and testing]
            - [Lack of governance]
            - [Network congestion and latency]
            - [Data integrity]
            - [Management]
            - [Versioning]
            - [Skillset]
        - [Best practices]
    - [Event-driven architecture style]
        - [When to use this architecture]
        - [Benefits]
        - [Challenges]
    - [Big data architecture style]
        - [When to use this architecture]
        - [Benefits]
            - [Technology choices]
            - [Performance through parallelism]
            - [Elastic scale]
            - [Interoperability with existing solutions]
        - [Challenges]
            - [Complexity]
            - [Skillset]
            - [Technology maturity]
            - [Security]
        - [Best practices]
            - [Leverage parallelism]
            - [Partition data]
            - [Apply schema-on-read semantics]
            - [Process data in-place]
            - [Balance utilization and time costs]
            - [Separate cluster resources]
            - [Orchestrate data ingestion]
            - [Scrub sensitive data early]
        - [IoT architecture]
    - [Big compute architecture style]
        - [Characteristics]
        - [When to use this architecture]
        - [Benefits]
        - [Challenges]
        - [Recommedation for Big compute using Azure Batch]
        - [Recommedation for Big compute running on Virtual Machines]
        - [Recommedation for HPC Pack deployed to Azure]
        - [Recommedation for Burst an HPC cluster to Azure]

## Architecture styles
[Architecture styles]: #architecture-styles
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
    
## Different type of styles
[Different type of styles]: #different-type-of-styles
1. N-tier architecture style
2. Web-Queue-Worker architecture style
3. Microservices architecture style
4. Event-driven architecture style
5. Big data architecture style
6. Big compute architecture style
    
### N-tier architecture style
[N-tier architecture style]: #n-tier-architecture-style
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
[When to use this architecture]: #when-to-use-this-architecture
- N-tier is best fit for migrating existing applications that already use a layered architecture.
- N-tier is most often seen in **infrastructure as a service (IaaS)** solutions, or application that use a mix of Iaas and managed services.
- N-tier architecture can be considered for following applications:
    - Simple web applications
    - Migrating an on-premises application to Azure with minimal refactoring
    - Unified development of on-premises and cloud applications.

#### Benefits
[Benefits]: #benefits
- Portability between cloud and on-premises, and between cloud platforms.
- Less learning curve for most developers.
- Natural evolution from the traditional application model.
- Open to heterogeneous environment (Windows/Linux)

#### Challenges
[Challenges]: #challenges
- It's easy to end up with a middle tier that just does CRUD operations on the database, adding extra latency without doing any useful work.
- Monolithic design prevents independent deployment of features.
- Managing an IaaS application is more work than an application that uses only managed services.
- It can be difficult to manage network security in a large system.

#### Best practices
[Best practices]: #best-practices
- Use autoscaling to handle changes in load.
- Use asynchronous messaging to decouple tiers.
- Cache semistatic data.
- Configure the database tier for high availability, using a solution such as SQL Server Always On availability groups.
- Place a web application firewall (WAF) between the front end and the Internet.
- Place each tier in its own subnet, and use subnets as a security boundary.
- Restrict access to the data tier, by allowing requests only from the middle tier(s).

#### Additional considerations
[Additional considerations]: #additional-considerations
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
[Recommendation for N-tier architecture on virtual machines]: #recommendation-for-n-tier-architecture-on-virtual-machines
![](/images/n_tier_vms.PNG)

- Each tier consists of two or more VMs, placed in an availability set or virtual machine scale set.
- Multiple VMs provide resiliency in case one VM fails.
- Load balancers are used to distribute requests across the VMs in a tier.
- A tier can be scaled horizontally by adding more VMs to the pool.
- Each tier is also placed inside its own subnet, meaning their internal IP addresses fall within the same address range and making it easy to apply network security group (NSG) rules and route tables to individual tiers.
- The web and business tiers are stateless.Any VM can handle any request for that tier.
- The data tier should consist of a replicated database. For Windows, we recommend SQL Server, using Always On availability groups for high availability. For Linux, choose a database that supports replication, such as Apache Cassandra.
- Network security groups restrict access to each tier.(For example, the database tier only allows access from the business tier.)

### Web-Queue-Worker architecture style
[Web-Queue-Worker architecture style]: #web-queue-worker-architecture-style
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
[When to use this architecture]: #when-to-use-this-architecture
- The Web-Queue-Worker architecture is typically implemented using managed compute services, either Azure App Service or Azure Cloud Services.
- Web-Queue-Worker architecture can be considered for following applications:
    - Applications with a relatively simple domain.
    - Applications with some long-running workflows or batch operations.
    - When you want to use managed services, rather than **infrastructure as a service (IaaS)**.

#### Benefits
[Benefits]: #benefits
- Relatively simple architecture that is easy to understand.
- Easy to deploy and manage.
- Clear separation of concerns.
- The front end is decoupled from the worker using asynchronous messaging.
- The front end and the worker can be scaled independently.

#### Challenges
[Challenges]: #challenges
- Without careful design, the front end and the worker can become large, monolithic components that are difficult to maintain and update.
- There may be hidden dependencies, if the front end and worker share data schemas or code modules.

#### Best practices
[Best practices]: #best-practices
- Expose a well-designed API to the client.
- Autoscale to handle changes in load.
- Cache semi-static data.
- Use a CDN to host static content.
- Use polyglot persistence when appropriate.
- Partition data to improve scalability, reduce contention, and optimize performance.

#### Additional considerations
[Additional considerations]: #additional-considerations
- Not every transaction has to go through the queue and worker to storage. The web front end can perform simple read/write operations directly. Workers are designed for resource-intensive tasks or long-running workflows. In some cases, you might not need a worker at all.
- Use the built-in autoscale feature of App Service to scale out the number of VM instances. If the load on the application follows predictable patterns, use schedule-based autoscale. If the load is unpredictable, use metrics-based autoscaling rules.
- Consider putting the web app and the function app into separate App Service plans. That way, they can be scaled independently.
- Use separate App Service plans for production and testing. Otherwise, if you use the same plan for production and testing, it means your tests are running on your production VMs.
- Use deployment slots to manage deployments. This lets you to deploy an updated version to a staging slot, then swap over to the new version. It also lets you swap back to the previous version, if there was a problem with the update.

#### Recommendation for Web-Queue-Worker on Azure App Service
[Recommendation for Web-Queue-Worker on Azure App Service]: #recommendation-for-web-queue-worker-on-azure-app-service

![](/images/web_queue_worker_azure_app_service.png)

- The front end is implemented as an Azure App Service web app.
- The worker is implemented as an Azure Functions app.
- The web app and the function app are both associated with an App Service plan that provides the VM instances.
- We can use either Azure Service Bus or Azure Storage queues for the message queue.
- Azure Cache for Redis stores session state and other data that needs low latency access.
- Azure Content Delivery Network (CDN) is used to cache static content such as images, CSS, or HTML.
- For storage, choose the storage technologies that best fit the needs of the application.
- You might use multiple storage technologies (polyglot persistence).

### Microservices architecture style
[Microservices architecture style]: #microservices-architecture-style
- A microservices architecture consists of a collection of small, autonomous services. Each service is self-contained and should implement a single business capability.

![](/images/microservices.png)

#### What are microservices?
[What are microservices?]: #what-are-microservices?
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
[Benefits]: #benefits
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
[Challenges]: #challenges
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
[Best practices]: #best-practices
- Model services around the business domain.
- Decentralize everything. Individual teams are responsible for designing and building services. Avoid sharing code or data schemas.
- Data storage should be private to the service that owns the data. Use the best storage for each service and data type.
- Services communicate through well-designed APIs. Avoid leaking implementation details. APIs should model the domain, not the internal implementation of the service.
- Avoid coupling between services. Causes of coupling include shared database schemas and rigid communication protocols.
- Offload cross-cutting concerns, such as authentication and SSL termination, to the gateway.
- Keep domain knowledge out of the gateway. The gateway should handle and route client requests without any knowledge of the business rules or domain logic. Otherwise, the gateway becomes a dependency and can cause coupling between services.
- Services should have loose coupling and high functional cohesion. Functions that are likely to change together should be packaged and deployed together. If they reside in separate services, those services end up being tightly coupled, because a change in one service will require updating the other service. Overly chatty communication between two services may be a symptom of tight coupling and low cohesion.
- Isolate failures. Use resiliency strategies to prevent failures within a service from cascading.

### Event-driven architecture style
[Event-driven architecture style]: #event-driven-architecture-style
- An event-driven architecture consists of event producers that generate a stream of events, and event consumers that listen for the events.

![](/images/event_driven.png)

- Events are delivered in near real time, so consumers can respond immediately to events as they occur.
- Producers are decoupled from consumers — a producer doesn't know which consumers are listening.
- Consumers are also decoupled from each other, and every consumer sees all of the events.
- This differs from a Competing Consumers pattern, where consumers pull messages from a queue and a message is processed just once (assuming no errors).
- An event driven architecture can use a pub/sub model or an event stream model.
    - *Pub/sub*
        - The messaging infrastructure keeps track of subscriptions.
        - When an event is published, it sends the event to each subscriber.
        - After an event is received, it cannot be replayed, and new subscribers do not see the event.
    - *Event streaming*
        - Events are written to a log.
        - Events are strictly ordered (within a partition) and durable.
        - Clients don't subscribe to the stream, instead a client can read from any part of the stream. 
        - The client is responsible for advancing its position in the stream. That means a client can join at any time, and can replay events.
- On the consumer side, there are some common variations:
    - *Simple event processing*
        - An event immediately triggers an action in the consumer. e.g. Use Azure Functions with a Service Bus trigger, so that a function executes whenever a message is published to a Service Bus topic.
    - *Complex event processing*
        - A consumer processes a series of events, looking for patterns in the event data, using a technology such as Azure Stream Analytics or Apache Storm. e.g. We could aggregate readings from an embedded device over a time window, and generate a notification if the moving average crosses a certain threshold.
    - *Event stream processing*
        - Use a data streaming platform, such as Azure IoT Hub or Apache Kafka, as a pipeline to ingest events and feed them to stream processors.
        - The stream processors act to process or transform the stream.
        - There may be multiple stream processors for different subsystems of the application. 
        - This approach is a good fit for IoT workloads.
#### When to use this architecture
[When to use this architecture]: #when-to-use-this-architecture
- Multiple subsystems must process the same events.
- Real-time processing with minimum time lag.
- Complex event processing, such as pattern matching or aggregation over time windows.
- High volume and high velocity of data, such as IoT.

#### Benefits
[Benefits]: #benefits
- Producers and consumers are decoupled.
- No point-to-point integrations. It's easy to add new consumers to the system.
- Consumers can respond to events immediately as they arrive.
- Highly scalable and distributed.
- Subsystems have independent views of the event stream.

#### Challenges
[Challenges]: #challenges
- Guaranteed delivery. In some systems, especially in IoT scenarios, it's crucial to guarantee that events are delivered.
- Processing events in order or exactly once. Each consumer type typically runs in multiple instances, for resiliency and scalability. This can create a challenge if the events must be processed in order (within a consumer type), or if the processing logic is not idempotent.

### Big data architecture style
[Big data architecture style]: #big-data-architecture-style
- A big data architecture is designed to handle the ingestion, processing, and analysis of data that is too large or complex for traditional database systems.

![](/images/big_data.png)

- Big data solutions typically involve one or more of the following types of workload
    - Batch processing of big data sources at rest.
    - Real-time processing of big data in motion.
    - Interactive exploration of big data.
    - Predictive analytics and machine learning.
   
- Some of the components of big data architectures:
    - *Data sources*
        - All big data solutions start with one or more data sources.
          E.g. Application data stores, such as relational databases; Static files produced by applications, such as web server log files ; Real-time data sources, such as IoT devices.
    - *Data storage*
        - Data for batch processing operations is typically stored in a distributed file store that can hold high volumes of large files in various formats. This kind of store is often called a data lake. 
        - E.g. Azure Data Lake Store or blob containers in Azure Storage
    - *Batch processing*
        - The data sets are so large, often a big data solution must process data files using long-running batch jobs to filter, aggregate, and otherwise prepare the data for analysis. Usually these jobs involve reading source files, processing them, and writing the output to new files.
        - E.g. Running U-SQL jobs in Azure Data Lake Analytics, using Hive, Pig, or custom Map/Reduce jobs in an HDInsight Hadoop cluster, or using Java, Scala, or Python programs in an HDInsight Spark cluster.
    - *Real-time message ingestion*
        - If the solution includes real-time sources, the architecture must include a way to capture and store real-time messages for stream processing. 
        - This might be a simple data store, where incoming messages are dropped into a folder for processing. 
        - However, many solutions need a message ingestion store to act as a buffer for messages, and to support scale-out processing, reliable delivery, and other message queuing semantics. 
        - E.g. Azure Event Hubs, Azure IoT Hubs, and Kafka.
    - *Stream processing*
        - After capturing real-time messages, the solution must process them by filtering, aggregating, and otherwise preparing the data for analysis. 
        - The processed stream data is then written to an output sink. 
        - Azure Stream Analytics provides a managed stream processing service based on perpetually running SQL queries that operate on unbounded streams.
        - E.g. Apache streaming technologies like Storm and Spark Streaming in an HDInsight cluster
    - *Analytical data store*
        - Many big data solutions prepare data for analysis and then serve the processed data in a structured format that can be queried using analytical tools. 
        - The analytical data store used to serve these queries can be a Kimball-style relational data warehouse, as seen in most traditional business intelligence (BI) solutions. 
        - Alternatively, the data could be presented through a low-latency NoSQL technology such as HBase, or an interactive Hive database that provides a metadata abstraction over data files in the distributed data store. 
        - E.g. Azure Synapse Analytics provides a managed service for large-scale, cloud-based data warehousing; HDInsight supports Interactive Hive, HBase, and Spark SQL, which can also be used to serve data for analysis.
    - *Analysis and reporting*
        - The goal of most big data solutions is to provide insights into the data through analysis and reporting. 
        - To empower users to analyze the data, the architecture may include a data modeling layer, such as a multidimensional OLAP cube or tabular data model in Azure Analysis Services. 
        - It might also support self-service BI, using the modeling and visualization technologies in Microsoft Power BI or Microsoft Excel. Analysis and reporting can also take the form of interactive data exploration by data scientists or data analysts. 
        - For these scenarios, many Azure services support analytical notebooks, such as Jupyter, enabling these users to leverage their existing skills with Python or R. 
        - For large-scale data exploration, you can use Microsoft R Server, either standalone or with Spark.
    - *Orchestration*
        - Most big data solutions consist of repeated data processing operations, encapsulated in workflows, that transform source data, move data between multiple sources and sinks, load the processed data into an analytical data store, or push the results straight to a report or dashboard. 
        - To automate these workflows, you can use an orchestration technology such Azure Data Factory or Apache Oozie and Sqoop.
        
Azure includes manay services that can be used in a big data architecture. They fall roughly into 2 categories:
1. Managed services
   e.g. Azure Data Lake Store, Azure Data Lake Analytics, Azure Synapse Analytics, Azure Stream Analytics, Azure Event Hub, Azure IoT Hub, and Azure Data Factory
2. Open source technologies based on the Apache Hadoop platform
   e.g. HDFS, HBase, Hive, Pig, Spark, Storm, Oozie, Sqoop, and Kafka. These technologies are available on Azure in the Azure HDInsight service

These options are not mutually exclusive, and many solutions combine open source technologies with Azure services.

#### When to use this architecture
[When to use this architecture]: #when-to-use-this-architecture
- Store and process data in volumes too large for a traditional database
- Transform unstructured data for analysis and reporting
- Capture, process, and analyze unbounded streams of data in real time, or with low latency
- Use Azure Machine Learning or Microsoft Cognitive Services

#### Benefits
[Benefits]: #benefits
- *Technology choices*
    - You can mix and match Azure managed services and Apache technologies in HDInsight clusters, to capitalize on existing skills or technology investments.
- *Performance through parallelism*
    - Big data solutions take advantage of parallelism, enabling high-performance solutions that scale to large volumes of data.
- *Elastic scale*
    - All of the components in the big data architecture support scale-out provisioning, so that you can adjust your solution to small or large workloads, and pay only for the resources that you use.
- *Interoperability with existing solutions*
    - The components of the big data architecture are also used for IoT processing and enterprise BI solutions, enabling you to create an integrated solution across data workloads.

#### Challenges
[Challenges]: #challenges
- *Complexity*
    - Big data solutions can be extremely complex, with numerous components to handle data ingestion from multiple data sources.
    - Challenging to build, test, and troubleshoot big data processes.
    - There may be a large number of configuration settings across multiple systems that must be used in order to optimize performance
- *Skillset*ata technologies are highly specialized, and use frameworks and languages that are not typical of more general application architectures.
      e.g. U-SQL language in Azure Data Lake Analytics is based on a combination of Transact-SQL and C#;SQL-based APIs are available for Hive, HBase, and Spark 
- *Technology maturity*
    - While core Hadoop technologies such as Hive and Pig have stabilized, emerging technologies such as Spark introduce extensive changes and enhancements with each new release.
    - Many big d
    - Managed services such as Azure Data Lake Analytics and Azure Data Factory are relatively young
- *Security*
    - Big data solutions usually rely on storing all static data in a centralized data lake.
    - Securing access to this data can be challenging, especially when the data must be ingested and consumed by multiple applications and platforms.

#### Best practices
[Best practices]: #best-practices
- *Leverage parallelism*
    - Most big data processing technologies distribute the workload across multiple processing units. 
    - This requires that static data files are created and stored in a splittable format. 
    - Distributed file systems such as HDFS can optimize read and write performance, and the actual processing is performed by multiple cluster nodes in parallel, which reduces overall job times.

- *Partition data*
    - Batch processing usually happens on a recurring schedule — for example, weekly or monthly. 
    - Partition data files, and data structures such as tables, based on temporal periods that match the processing schedule. 
    - That simplifies data ingestion and job scheduling, and makes it easier to troubleshoot failures. 
    - Also, partitioning tables that are used in Hive, U-SQL, or SQL queries can significantly improve query performance.

- *Apply schema-on-read semantics*
    - Using a data lake lets you to combine storage for files in multiple formats, whether structured, semi-structured, or unstructured. 
    - Use schema-on-read semantics, which project a schema onto the data when the data is processing, not when the data is stored. 
    - This builds flexibility into the solution, and prevents bottlenecks during data ingestion caused by data validation and type checking.

- *Process data in-place*
    - Traditional BI solutions often use an extract, transform, and load (ETL) process to move data into a data warehouse. 
    - With larger volumes data, and a greater variety of formats, big data solutions generally use variations of ETL, such as transform, extract, and load (TEL). 
    - With this approach, the data is processed within the distributed data store, transforming it to the required structure, before moving the transformed data into an analytical data store.
- *Balance utilization and time costs*
    - For batch processing jobs, it's important to consider two factors: 
        - The per-unit cost of the compute nodes 
        - The per-minute cost of using those nodes to complete the job. 
          e.g. a batch job may take eight hours with four cluster nodes. However, it might turn out that the job uses all four nodes only during the first two hours, and after that, only two nodes are required. In that case, running the entire job on two nodes would increase the total job time, but would not double it, so the total cost would be less. 
     - In some business scenarios, a longer processing time may be preferable to the higher cost of using underutilized cluster resources.

- *Separate cluster resources*
    - When deploying HDInsight clusters, you will normally achieve better performance by provisioning separate cluster resources for each type of workload. 
      For example, although Spark clusters include Hive, if you need to perform extensive processing with both Hive and Spark, you should consider deploying separate dedicated Spark and Hadoop clusters. 
    - Similarly, if you are using HBase and Storm for low latency stream processing and Hive for batch processing, consider separate clusters for Storm, HBase, and Hadoop.

- *Orchestrate data ingestion*
    - Existing business applications may write data files for batch processing directly into Azure storage blob containers, where they can be consumed by HDInsight or Azure Data Lake Analytics. 
    - We will often need to orchestrate the ingestion of data from on-premises or external data sources into the data lake. 
    - Use an orchestration workflow or pipeline, such as those supported by Azure Data Factory or Oozie, to achieve this in a predictable and centrally manageable fashion.

- *Scrub sensitive data early*
    - The data ingestion workflow should scrub sensitive data early in the process, to avoid storing it in the data lake.

#### IoT architecture
[IoT architecture]: #iot-architecture
Internet of Things (IoT) is a specialized subset of big data solutions.

![Event-streaming components of the architecture](/images/Iot_architecture.png)

- The cloud gateway ingests device events at the cloud boundary, using a reliable, low latency messaging system.
- Devices might send events directly to the cloud gateway, or through a field gateway.
- A field gateway is a specialized device or software, usually colocated with the devices, that receives events and forwards them to the cloud gateway.
- The field gateway might also preprocess the raw device events, performing functions such as filtering, aggregation, or protocol transformation.
- After ingestion, events go through one or more stream processors that can route the data (for example, to storage) or perform analytics and other processing.
- Common type of processing:
    - Writing event data to cold storage, for archiving or batch analytics.
    - Hot path analytics, analyzing the event stream in (near) real time, to detect anomalies, recognize patterns over rolling time windows, or trigger alerts when a specific condition occurs in the stream.
    - Handling special types of non-telemetry messages from devices, such as notifications and alarms.
    - Machine learning.
- These components of an IoT system do not directly related to event streaming:
    - Device registry: Database of the provisioned devices, including the device IDs and usually device metadata, such as location.
    
    - Provisioning API: It is a common external interface for provisioning and registering new devices.
    
    - Command and control messages: Allow command and control messages to be sent to devices.

### Big compute architecture style
[IoT architecture]: #iot-architecture

#### When to use this architecture
[When to use this architecture]: #when-to-use-this-architecture
- Big compute describes large-scale workloads that require a large number of cores, often numbering in the hundreds or thousands. 
- Scenarios include image rendering, fluid dynamics, financial risk modeling, oil exploration, drug design, and engineering stress analysis, etc.

![](/images/big_compute.png)

#### Characteristics
[Characteristics]: #characteristics
- The work can be split into discrete tasks, which can be run across many cores simultaneously.
- Each task is finite. 
- It takes some input, does some processing, and produces output.
- The entire application runs for a finite amount of time (minutes to days).
- A common pattern is to provision a large number of cores in a burst, and then spin down to zero once the application completes.
- The application does not need to stay up 24/7.
- The system must handle node failures or application crashes.
- For some applications, tasks are independent and can run in parallel.
- Also there are tasks are tightly coupled, meaning they must interact or exchange intermediate results.In that case, consider using high-speed networking technologies such as InfiniBand and remote direct memory access (RDMA).
- Depending on your workload, you might use compute-intensive VM sizes (H16r, H16mr, and A9).

#### When to use this architecture
[When to use this architecture]: #when-to-use-this-architecture
- Computationally intensive operations such as simulation and number crunching.
- Simulations that are computationally intensive and must be split across CPUs in multiple computers (10-1000s).
- Simulations that require too much memory for one computer, and must be split across multiple computers.
- Long-running computations that would take too long to complete on a single computer.
- Smaller computations that must be run 100s or 1000s of times, such as Monte Carlo simulations.

#### Benefits
[Benefits]: #benefits
- High performance with "embarrassingly parallel" processing.
- Can harness hundreds or thousands of computer cores to solve large problems faster.
- Access to specialized high-performance hardware, with dedicated high-speed InfiniBand networks.
- You can provision VMs as needed to do work, and then tear them down.

#### Challenges
[Challenges]: #challenges
- Managing the VM infrastructure.
- Managing the volume of number crunching
- Provisioning thousands of cores in a timely manner.
- For tightly coupled tasks, adding more cores can have diminishing returns. You may need to experiment to find the optimum number of cores.

#### Recommedation for Big compute using Azure Batch
[Recommedation for Big compute using Azure Batch]: #recommedation-for-big-compute-using-azure-batch
Azure Batch is a managed service for running large-scale high-performance computing (HPC) applications.

![](/images/big_compute_azure_batch.png)

- Using Azure Batch, you configure a VM pool, and upload the applications and data files.
- Then the Batch service provisions the VMs, assign tasks to the VMs, runs the tasks, and monitors the progress.
- Batch can automatically scale out the VMs in response to the workload.
- Batch also provides job scheduling.

#### Recommedation for Big compute running on Virtual Machines
[Recommedation for Big compute running on Virtual Machines]: #recommedation-for-big-compute-running-on-virtual-machines
Microsoft HPC Pack is used to administer a cluster of VMs, and schedule and monitor HPC jobs.

- We must provision and manage the VMs and network infrastructure.
- This approach have existing HPC workloads and want to move some or all it to Azure.
- You can move the entire HPC cluster to Azure, or you can keep your HPC cluster on-premises but use Azure for burst capacity.

#### Recommedation for HPC Pack deployed to Azure
[Recommedation for HPC Pack deployed to Azure]: #recommedation-for-hpc-pack-deployed-to-azure
The HPC cluster is created entirely within Azure.

![](/images/hpc_pack_azure.png)

- The head node provides management and job scheduling services to the cluster.
- For tightly coupled tasks, use an RDMA network that provides very high bandwidth, low latency communication between VMs.

#### Recommedation for Burst an HPC cluster to Azure
[Recommedation for Burst an HPC cluster to Azure]: #recommedation-for-burst-an-hpc-cluster-to-azure
An organization is running HPC Pack on-premises, and uses Azure VMs for burst capacity.

![](/images/burst_hpc_cluster_azure.png)

- The cluster head node is on-premises.
- ExpressRoute or VPN Gateway connects the on-premises network to the Azure VNet.
