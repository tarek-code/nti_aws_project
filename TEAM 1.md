# **Team 1  moghazy , tarek , ali , anas , nour eldeen 0**

# **Project: Design a Highly Available Three-Tier VPC Architecture**

Company: GlobalBook Inc.  
 **Scenario:**  
 GlobalBook Inc. is launching a new e-commerce platform for selling books online. Their application is structured as a three-tier architecture:  
 • Presentation Tier: Web servers that serve the website to users.  
 • Application Tier: Application servers that handle the core business logic  
 • Data Tier: A database that stores all product, user, and transaction data.  
 The application must be designed for high availability to survive infrastructure failures and scalability to handle unpredictable traffic loads.

**Your Task:**  
 You are an AWS Solutions Architect. Design the VPC network architecture that will host this application. Your design must meet the following core objectives:

1. High Availability (HA): The infrastructure must be resilient to failure.  
2. Scalability: Each tier must be able to scale out independently based on load.  
3. Security: The architecture must follow security best practices

**Technical Requirements:**

 • The Presentation Tier must be accessible to users on the public internet.  
 • The Application Tier must be private and only accept traffic from the Presentation Tier.  
 • The Data Tier must be private and only accept traffic from the Application Tier.  
 • Servers in private subnets (App & Data Tiers) need secure, outbound access to the internet for updates and patches.  
 • The database must be deployed in a high-availability configuration.  
 

## Expected Deliverables:

1. Scope of Work (SoW)  
2. A High-Level Architecture Diagram illustrating the VPC, subnets in multiple AZs, placement of all components, and traffic flow.  
3. Technical details: A brief written explanation justifying your design choices for high availability and scalability.  
4. Cost estimation for all architecture  
5. size estimation (instance types, storage size/tier, expected bandwidth considerations).  
