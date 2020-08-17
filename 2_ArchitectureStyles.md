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
- Dependencies are managed by dividing the application into layers such as presentation, business logic, and data access.

  ![](/images/n_tier.png)

- A layer can only call into layers that sit below it.
- Horizontal layering a liability, it can be hard to introduce changes in one part of the application without touching the rest of the application. Frequent updates are challenging, limiting how quickly new features can be added.
- N-tier is best fit for migrating existing applications that already use a layered architecture.
- N-tier is most often seen in **infrastructure as a service (IaaS)** solutions, or application that use a mix of Iaas and managed services.
