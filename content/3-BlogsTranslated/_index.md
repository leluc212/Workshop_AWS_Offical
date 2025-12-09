---
title: "Translated Blogs"
date: "2025-09-08"
weight: 3
chapter: false
pre: " <b> 3. </b> "
---



###  [Blog 1 - Cost Optimization in Cloud Computing: Rehost Migration Handbook (Part 4 – Migration: Realizing Cost Savings)](3.1-Blog1/)
This blog is part 4 of a four-part series on cost optimization during Rehost Migration to AWS, focusing on the migrate phase - realizing the planned cost savings. The article introduces 4 main AWS tools for post-migration cost optimization: AWS Billing and Cost Management (monitoring and analyzing overall costs), AWS Cost Explorer (visualizing spending trends), AWS Cost Optimization Hub (providing optimization recommendations such as switching to Graviton instances, rightsizing, removing unused resources), and AWS Compute Optimizer (suggesting rightsizing for EC2, EBS, Lambda, RDS based on actual metrics). The goal is to help businesses not only achieve initial savings but also maintain continuous cost optimization through regular analysis and implementation of appropriate rightsizing recommendations.

###  [Blog 2 - Exploring AWS Console: Diagnosing Errors with Amazon Q Developer](3.2-Blog2/)
This blog introduces the "Diagnose with Amazon Q" feature of Amazon Q Developer - an AI assistant that helps diagnose and troubleshoot errors directly in the AWS Management Console. When encountering an error, users simply click the "Diagnose with Amazon Q" button, and the system uses LLM (Large Language Models) to analyze the root cause based on contextual information (error message, URL, IAM role). The "Help me resolve" feature then provides detailed step-by-step remediation guidance. Amazon Q operates by: collecting contextual information from the console, proactively querying additional data via AWS Cloud Control API (within the user's IAM permissions), and synthesizing it into specific solutions. The goal is to reduce MTTR (Mean Time To Repair), save troubleshooting time, and help developers/operators handle incidents more efficiently without navigating across multiple consoles.

###  [Blog 3 - Getting Started with Apache OFBiz and Amazon Aurora DSQL](3.3-Blog3/)
This blog presents a real-world case study of migrating Apache OFBiz (open-source ERP/CRM with 837 tables, 4161 indexes) from PostgreSQL to Amazon Aurora DSQL. Key changes include: (1) Implementing IAM-based authentication instead of password authentication, with time-limited tokens and connection pooling management; (2) Schema adjustments - changing NUMERIC data types to BIGINT, reducing VARCHAR size in keys due to current Aurora DSQL limitations; (3) Handling transaction limits - splitting batch data loading into batches of 1000 rows per commit; (4) Handling OCC (Optimistic Concurrency Control) - adding retry logic for transactions that may abort due to conflicts, as Aurora DSQL only supports snapshot isolation and doesn't lock resources during transaction building. The results demonstrate that Aurora DSQL can support complex applications with reasonable migration effort, focusing on 4 key aspects: using supported data types, IAM authentication, transaction size limits, and idempotent retry-able transactions.

