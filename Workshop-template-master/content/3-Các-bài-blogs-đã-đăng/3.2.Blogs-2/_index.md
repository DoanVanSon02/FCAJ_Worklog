---
title : "Blog Posts"
date :  05-11-2026
weight : 2
chapter : false
pre : " <b> 3.2 </b> "
---

# 📌 Planning and measuring EC2 capacity fulfillment with Amazon EC2 Capacity Manager

Hi everyone, I recently read **“Plan and measure EC2 flexibility with Amazon EC2 Capacity Manager”** by **Yagya Vir Singh and Brian Phillippi**, published on the AWS Compute Blog on **November 4, 2026**.

The article raises an interesting question: When a system needs to scale, how do we know which EC2 instance types and Availability Zones to choose? After deployment, how do we check whether those choices help the system receive enough resources?

I have put together what I learned to discuss with everyone. The figures below come from the illustrative AnyCompany scenario in the original AWS article, not from my own testing. [1]

## 🔎 First, what is Amazon EC2?

**Amazon EC2**, short for **Amazon Elastic Compute Cloud**, is a service that provides on-demand computing resources on AWS. Simply put, we launch virtual servers, called instances, to run applications and increase or decrease resources as needed.

Each instance type offers a different resource configuration, such as CPU, memory, networking capabilities, and storage, to suit different usage requirements. [2]

## 🔎 What is Amazon EC2 Capacity Manager?

**Amazon EC2 Capacity Manager** is a tool that helps monitor, analyze, and manage EC2 capacity from a centralized interface, including On-Demand Instances, Spot Instances, and Capacity Reservations.

The tool supports aggregating data across multiple Regions and, when integrated with AWS Organizations, across member accounts in the organization. This allows operations teams to observe resource usage and identify optimization opportunities without manually consolidating data from multiple sources. [3]

In this article, AWS focuses on two capabilities:

- **Real-time supportability checks:** assess how well each instance type and Availability Zone pair can fulfill a request before scaling, without launching or reserving instances.
- **Capacity fulfillment metrics:** measure the resources actually delivered against the amount requested, while showing which instance types and Availability Zones supplied those resources.

These two capabilities are included with **AWS Unified Operations**. The distinction to keep in mind is that fulfillment metrics measure actual launch results, not the accuracy of supportability checks. [1][4]

![Figure 1 – Overview of the two capabilities, source: AWS](https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2026/11/04/ComputeBlog-2823-1-1.png)

## 💡 What does “flexibility” mean in this article?

Instead of restricting a workload to one instance type in one Availability Zone (AZ), we configure it to allow multiple suitable options. EC2 then has more ways to fulfill the request when one option does not have enough capacity.

In the article, a **“combination”** means an **instance type × AZ** pair, such as `r5.16xlarge` in `use1-az5`.

AWS illustrates how to assess and improve this flexibility through the following four steps. [1]

## 🛠️ Before you begin

- Enable Capacity Manager for a single account or at the organization level through the management account or a delegated administrator.
- Confirm that the account is covered by AWS Unified Operations to use the two capabilities discussed.
- To monitor metrics by workload, activate the corresponding tag as a monitored tag key. Some tags, such as `aws:eks:cluster-name` and `aws:autoscaling:groupName`, are already available and require no activation. The original article uses `aws:eks:cluster-name`.

You can query data from commercial AWS Regions through the Region where Capacity Manager is enabled. [1]

## 1️⃣ Check whether workloads are receiving enough resources

In Capacity Manager, open the **Fulfillment** tab, select a time range and the **On-Demand** or **Spot** metric set. Then filter by Region, account, or tag.

Four metrics to pay attention to:

- **`CapacityTotalRequestedVcpu`:** the total number of vCPUs requested. Retries within five minutes using the same configuration are counted only once.
- **`CapacityTotalFulfilledVcpu`:** the total number of vCPUs actually delivered.
- **`CapacityRequestFulfillmentRate`:** the request fulfillment rate, calculated as delivered vCPUs divided by requested vCPUs.
- **`AvgInstAzCountPerRequestedVcpu`:** the average number of instance type × AZ combinations that requests are configured to use. A value greater than 1 indicates that the requests are flexible.

We can sort by **Requested** to see the workloads with the highest demand, or by **request fulfillment rate** to identify workloads that need improvement.

In the AWS example:

- **search-index** requested **51,712 vCPUs** and received **45,632 vCPUs**, corresponding to an **88.2%** fulfillment rate; its requests used an average of **21 combinations**.
- **batch-etl** could use only **one combination** and had a **72.8%** fulfillment rate.

This is the starting point for identifying workloads that are short on capacity and need further analysis. When using the CLI, the original article queries the data through `get-capacity-manager-metric-data`. [1]

![Figure 2 – Fulfillment by workload, source: AWS](https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2026/11/04/ComputeBlog-2823-2.png)

## 2️⃣ See where flexibility is paying off

On a row in the **Fulfillment** tab, select **View breakdown** to see details by instance type, instance family, and AZ.

One point I think deserves careful attention is that the fulfillment rate for the overall request and the rate for each combination are calculated at two different levels. When a request has multiple options, one combination can compensate for another that lacks capacity.

For **search-index**:

- The three combinations in `use1-az5` fulfilled approximately **68.6–82.8%**.
- `r5.16xlarge` in `use1-az4` and `use1-az6` fulfilled **98.5–100%**.
- `r6a.16xlarge` in `use1-az4` fulfilled **100%**.

This means the other AZs compensated for part of the shortfall in `use1-az5`.

In the original article's breakdown, the **28 combinations** had an average individual fulfillment rate of **46.6%**, while the request-level rate reached **88.2%**. This illustrates the benefit of having multiple options for obtaining resources.

However, `use1-az5` still delivered **26,880 vCPUs**, more than half of the total delivered capacity. Therefore, this is where additional options or capacity reservations should be considered, rather than simply removing it because of its lower rate.

Each row also has a **Check supportability** link to check the corresponding combination. When querying the `InstAzFulfillmentRate` metric through the API, the `availability-zone-id` dimension and either `instance-type` or `instance-family` are required. [1]

![Figure 3 – Comparing request-level and per-combination rates, source: AWS](https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2026/11/04/ComputeBlog-2823-3.png)

![Figure 4 – Breakdown and Check supportability links, source: AWS](https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2026/11/04/ComputeBlog-2823-4.png)

## 3️⃣ Check supportability before scaling

Before a major scale-up or migration, open the **Supportability** tab and enter:

- The resource type: On-Demand or Spot.
- The amount of capacity required.
- The intended AZs.
- A list of instance types or resource attribute requirements.

The results include a score for each combination and an aggregate score:

- **High:** the combination has capacity exceeding the request by a substantial margin.
- **Medium:** it can fulfill the request, but with little capacity to spare.
- **Low:** capacity is below the requested amount.

In the example, AnyCompany checks a requirement for **100 instances** across three AZs: `use1-az4`, `use1-az5`, and `use1-az6`, using five instance types: `r5.16xlarge`, `r6a.16xlarge`, `r7a.16xlarge`, `r6i.16xlarge`, and `r7i.16xlarge`.

The aggregate result is **High**, but the **15 combinations** have different results:

- **4 combinations** score High.
- **5 combinations** score Medium.
- **6 combinations** score Low.

Specifically, `r5.16xlarge` scores High in all three AZs; `r7a.16xlarge` scores High in `use1-az6`. `r6a.16xlarge` scores Medium in all three AZs, while `r7i.16xlarge` scores Medium in `use1-az4` and `use1-az6`. All combinations involving `r6i.16xlarge` score Low.

A single check supports up to **10 AZs**, which can span multiple Regions, and up to **100 combinations**. In addition to listing instance types, you can select them by attributes; the example in the article specifies **64 vCPUs** and a minimum memory size of **524,288 MiB**.

> Keep in mind: scores provide guidance at the time of the check, not a commitment to hold resources available. If capacity assurance is required, AWS points to Capacity Reservations or future-dated Capacity Reservations.

AWS also recommends checking during the planning stage rather than before every launch, as doing so could introduce additional cross-Region dependencies and consume the hourly quota. [1]

![Figure 5 – Checking a requirement for 100 instances, source: AWS](https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2026/11/04/ComputeBlog-2823-5.png)

## 4️⃣ Apply the results to the actual configuration

After identifying suitable options, update the **Auto Scaling group**, **EC2 Fleet**, or **Karpenter** configuration so that subsequent launches can benefit from that flexibility.

For the Auto Scaling group in the article, the team:

- Keeps `r5.16xlarge`, `r6a.16xlarge`, and `r7a.16xlarge`.
- Adds `r7i.16xlarge` because it scores Medium in two AZs.
- Does not add `r6i.16xlarge` because all its combinations score Low.

The example uses a **mixed instances policy** with the **“prioritized”** On-Demand strategy. For workloads described by vCPU and memory requirements, **attribute-based instance type selection** can be used to expand the set of suitable instances.

After the change, monitor `AvgInstAzCountPerRequestedVcpu` to check whether the number of options in the requests has increased.

If there is a fixed requirement in a specific AZ, you can combine reserving baseline capacity with keeping the remaining capacity flexible. The original article illustrates reserving **10 r5.16xlarge instances in use1-az5**.

One notable detail: `r5.16xlarge` in this AZ previously fulfilled only **82.8%**, even though the current check gives it a **High** score. The two results reflect historical data and the state at the time of the check, so they do not contradict each other.

**Capacity Reservations are still charged even when unused**, so the reserved amount should be appropriate. For the reservation in the example, which has no commitment duration, cancellation incurs no cancellation fee; running instances continue operating at On-Demand rates. [1]

![Figure 6 – From assessment to fleet configuration, source: AWS](https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2026/11/04/ComputeBlog-2823-6.png)

## 📝 What I learned from the article

Allowing multiple instance types and AZs becomes more useful when we have data to assess how well each option works.

Supportability checks help inform decisions before scaling; fulfillment metrics help review the resources actually delivered. From there, we can identify where to add options and which portion needs to be reserved.

AWS suggests reviewing fulfillment rates weekly, checking the breakdown for workloads that fall short of their targets, and incorporating supportability checks into the change management process before adding or scaling workloads. [1]

**Have you ever encountered a situation where Auto Scaling needed to add instances but could not obtain enough capacity? In that case, do you usually prioritize adding instance types, expanding to another AZ, or combining these approaches with Capacity Reservations?** I would love to hear more about your practical experiences and feedback to improve my understanding 😊

## 📚 References

**[1] Yagya Vir Singh and Brian Phillippi — “Plan and measure EC2 flexibility with Amazon EC2 Capacity Manager”, AWS Compute Blog, November 4, 2026.** Source for the entire four-step workflow, the AnyCompany figures, and the six illustrations:

[https://aws.amazon.com/blogs/compute/plan-and-measure-ec2-flexibility-with-amazon-ec2-capacity-manager/](https://aws.amazon.com/blogs/compute/plan-and-measure-ec2-flexibility-with-amazon-ec2-capacity-manager/)

**[2] AWS Documentation — “What is Amazon EC2?”:**

[https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/)

**[3] AWS Documentation — “EC2 Capacity Manager”:**

[https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/capacity-manager.html](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/capacity-manager.html)

**[4] AWS What's New — “New features for Amazon EC2 Capacity Manager”, November 4, 2026:**

[https://aws.amazon.com/about-aws/whats-new/2026/11/ec2-capacity-manager-supportability-and-fulfillment/](https://aws.amazon.com/about-aws/whats-new/2026/11/ec2-capacity-manager-supportability-and-fulfillment/)

#AWS #AmazonEC2 #EC2CapacityManager #AWSStudyGroup #FCJ #CloudComputing