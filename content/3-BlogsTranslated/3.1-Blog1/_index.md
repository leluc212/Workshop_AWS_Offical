---
title: "Blog 1"
date: "2025-09-08"
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---


**Migration & Modernization**  
# Cost Optimization in Cloud Computing: Rehost Migration Handbook (Part 4 – Migration: Realizing Cost Savings)
**Authors:** Dan Krueger, James Gaines, and Rohit Vaitheeswaran – 31/03/2025  
**Categories:** AWS Cloud Financial Management, AWS Compute Optimizer, AWS Cost Explorer, Best Practices, Billing & Account Management, Cloud Cost Optimization, Customer Enablement, Enterprise Strategy, Thought Leadership  

---  

This article is the fourth part in a four-part step-by-step guide to optimizing costs throughout the Rehost migration process on AWS, specifically covering:  
- Discovering cost components and on-premises environment.  
- Building an accurate business case during the assess phase.  
- Understanding and controlling cloud spend during the mobilize phase.  
- Cost optimization to achieve planned financial savings during the migrate phase.  

---  

**Figure 1:** Cost activity overview of the Rehost Migration process by phase  

---  

## 1. Cost Optimization on AWS:  
We use specific AWS services and tools throughout the Rehost migration to optimize costs.  
These AWS services provide recommendations to optimize Amazon EC2 instance costs, storage costs, and overall application run costs. Additional savings may come from initial migration and optimization based on application performance metrics from the on-premises data center (including CPU, memory, disk I/O, and network). Post-migration cost optimization requires analyzing usage metrics, which vary by instance type and size.  

---  

### 1.1. AWS Billing and Cost Management Home  
AWS Billing and Cost Management provides essential services for cost optimization throughout the Rehost migration.  
If using AWS Organizations with a management account for centralized governance, configure member accounts’ access to these services per organizational policies. These services apply to many teams beyond designated cost management stakeholders. The AWS Billing and Cost Management console provides summaries enabling account owners to drill down into specific categories.  

The landing page includes default widgets offering a quick view of current forecast costs compared to recent months.  
The Info tab shows detailed summaries explaining each widget for the account owner.  

Default widgets include:  
- **Cost Summary:** Displays current spending trends versus the previous month. Costs shown exclude credits and discounts.  
- **Cost Monitor:** Shows costs, budgets, and AWS-detected anomalies based on Cost Anomaly Detection configurations.  
- **Cost Breakdown:** Displays cost analysis over the past six months, grouped by AWS service, member account, region, cost allocation tags, and cost categories.  
- **Recommended Actions:** Provides guidance for AWS account owners on cloud financial management best practices and cost optimization based on recommendations.  
- **Savings Opportunities:** Offers suggestions from the Cost Optimization Hub in various categories, such as resizing, instance type changes, and cleanup of unused resources.  

---  

### 1.2. AWS Cost Explorer  
The Cost Breakdown widget can be expanded to AWS Cost Explorer, allowing account owners and cost managers to visualize, analyze, and manage AWS spending.  
AWS Cost Explorer provides both overview and detailed views of spending trends. Users can filter and forecast to identify cost drivers and anomalies. Reports can be customized by time range (hour, day, month) and grouped by service. Saved reports can be stored for future use.  

---  

### 1.3. AWS Cost Optimization Hub  
The AWS Cost Optimization Hub offers an overview of cost optimization opportunities for migrated EC2 instances.  

**Figure 2:** Cost Optimization Hub and Recommendations  

After Rehost migration completes, the system recommends switching to Graviton instances for additional cost savings.  
Based on CPU and memory usage metrics, the hub suggests right-sizing instances and removing inactive resources.  

Leveraging the Cost Optimization Hub helps cost management teams identify, prioritize, and implement effective savings measures, improving cloud financial management and optimizing resources beyond savings achieved during migration.  

---  

### 1.4. AWS Compute Optimizer  
AWS Compute Optimizer reduces EC2 instance costs with size-appropriate recommendations. It provides savings overviews, performance improvements, and optimization advice by region for resources including:  
- Amazon EC2  
- Amazon EC2 Auto Scaling Groups  
- Amazon EBS  
- AWS Lambda functions  
- Amazon ECS on AWS Fargate  
- Commercial software licenses  
- Amazon RDS DB instances and storage  

When enabled, Compute Optimizer evaluates resources based on technical specifications and usage patterns recorded by Amazon CloudWatch over the past 14 days, considering CPU utilization, network traffic, disk activity, and current instance usage levels.  

---  

## 2. Conclusion  
Establish dedicated ownership and implement systematic cost monitoring processes using AWS Cost Explorer, AWS Budgets, and AWS Cost and Usage Reports.  
These tools, combined with AWS Cost Optimization Hub and AWS Compute Optimizer, provide actionable recommendations for EC2 instances, storage, and application optimization. Regular analysis of resource usage patterns and implementing resizing recommendations maintain long-term cost efficiency beyond initial migration savings, laying the foundation for continuous cost optimization.  

This article concludes the Rehost migration cost optimization journey during the migrate phase by reviewing:  
- 1.1 AWS Billing and Cost Management Home services,  
- 1.2 AWS Cost Explorer for visualization and cost analysis,  
- 1.3 AWS Cost Optimization Hub and  
- 1.4 AWS Compute Optimizer for detailed cost optimization recommendations and resizing opportunities.  

---  

## Blog Series  
Direct links to each part of the series:  
- Part 1: Assess – Explore Cost Components  
- Part 2: Assess – Build a Business Case  
- Part 3: Mobilize – Understand and Control Cloud Spend  
- Part 4: Migrate – Realizing Planned Savings  

---  

## Additional Resources  
Contact AWS migration experts to discuss how we can support your organization.  
Ready to migrate and optimize costs? Additional resources include:  
- AWS Cloud Financial Management Guide to adjust your financial processes for cloud readiness.  
- Latest AWS Cloud Financial Management content.  
- Learn more about migration and modernization with AWS.  
- Explore Amazon Q Developer — an AI-powered assistant helping transform .NET, mainframe, VMware, and Java workloads beyond traditional Rehost models.  

---  

## About the Authors  
**Dan Krueger** – Senior Customer Solutions Manager at Amazon Web Services, with extensive experience supporting U.S. Federal Government customers. At AWS, he has led large-scale cloud migration projects helping agencies enhance mission execution capabilities. Previously a Program Executive at IBM focused on data platform modernization and complex technology solutions for government clients.  

**James Gaines** – Senior Solutions Architect in Healthcare and Life Sciences at AWS. Experienced in highly regulated environments including the Department of Defense and pharmaceutical industry. Holds all AWS Certifications and specializes in cloud migrations, application modernization, and advanced analytics driving innovation in healthcare and life sciences.  

**Rohit Vaitheeswaran** – Senior Solutions Architect at AWS, specializing in Healthcare and Life Sciences. Experienced in multiple industries leading large-scale migration strategy and cloud optimization initiatives. Has helped organizations in Financial Services, Fintech, and Healthcare optimize their cloud journey on AWS with a focus on migration strategy and cost optimization, maximizing cloud investment and ensuring compliance with industry-specific requirements.
