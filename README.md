[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/IAASVEAZ)
# CSIT5970 Assignment-1: EC2 Measurement (2 questions, 4 marks)

### Deadline: 11:59PM, Feb, 28, Friday

---

### Name: Huang Jiaxiang
### Student Id: 21076325
### Email: jhuangeg@connect.ust.hk

---

## Question 1: Measure the EC2 CPU and Memory performance

1. (1 mark) Report the name of measurement tool used in your measurements (you are free to choose *any* open source measurement software as long as it can measure CPU and memory performance). Please describe your configuration of the measurement tool, and explain why you set such a value for each parameter. Explain what the values obtained from measurement results represent (e.g., the value of your measurement result can be the execution time for a scientific computing task, a score given by the measurement tools or something else).

    For this experiment, I use sysbench, an open-source benchmarking tool, to measure CPU and memory performance of different EC2 instances.

    #### CPU Performance Test
    ```
    sysbench cpu --cpu-max-prime=20000 run
    ```
    **--cpu-max-prime=20000:** This parameter specifies the maximum prime number to be calculated, which increases the computational workload. I choose 20000 to ensure that the CPU has a significant workload to process, making performance differences more noticeable.

   - Why?
     - A larger number increases CPU load, helping to distinguish performance differences between instance types.
     - The metric reported is "events per second", representing how many operations the CPU can complete per second. Higher values indicate better performance.

    #### Memory Performance Test
    ```
    sysbench memory --memory-block-size=1M --memory-total-size=10G run
    ```
    **--memory-block-size=1M:** This specifies that memory operations will be performed in 1MB chunks.

    **--memory-total-size=10G:** This tells sysbench to process 10GB of memory operations in total.
   - Why?
     - This ensures we are testing sustained memory performance over a large enough workload.
     - The result is typically reported in MB/s (megabytes per second), indicating the memory read/write speed. Higher values mean better memory performance.


1. (1 mark) Run your measurement tool on general purpose `t2.micro`, `t2.medium`, and `c5d.large` Linux instances, respectively, and find the performance differences among these instances. Launch all the instances in the **US East (N. Virginia)** region. Does the performance of EC2 instances increase commensurate with the increase of the number of vCPUs and memory resource?

    In order to answer this question, you need to complete the following table by filling out blanks with the measurement results corresponding to each instance type.

    | Size        | CPU performance (events/sec) | Memory performance (MB/s) |
    | ----------- | ---------------------------- | ------------------------- |
    | `t2.micro`  | 891.91                       | 19246.57                  |
    | `t2.medium` | 887.08                       | 18969.89                  |
    | `c5d.large` | 449.74                       | 19559.65                  |

    > Region: US East (N. Virginia). Use `Ubuntu Server 22.04 LTS (HVM)` as AMI.

    The CPU performance of t2.micro and t2.medium is nearly identical (891.91 vs. 887.08 events/sec), indicating that they share the same vCPU architecture. However, despite having more vCPUs, t2.medium does not show a significant advantage in single-threaded tests.

    The CPU performance of c5d.large is significantly lower (449.74 events/sec), likely due to its classification as a Compute Optimized instance. The default CPU scheduling strategy may have affected sysbench's single-threaded test results.

    The memory bandwidth of all three instances is around 19GB/s, with minimal variation, suggesting that RAM performance remains nearly identical across different instance types under default test conditions. Although c5d.large achieves the highest memory bandwidth at 19559.65 MB/s, the difference is not substantial, indicating that increasing vCPU count does not significantly improve EC2 instance memory speed.

## Question 2: Measure the EC2 Network performance
![network-output-sample](https://github.com/user-attachments/assets/913b1a19-69ce-4d73-9e0d-e27e7bf5c64c)
![ping-output-sample](https://github.com/user-attachments/assets/b4db9fc4-3540-4e38-94d0-44d102a51b94)

1. (1 mark) The metrics of network performance include **TCP bandwidth** and **round-trip time (RTT)**. Within the same region, what network performance is experienced between instances of the same type and different types? In order to answer this question, you need to complete the following table.

    | Type                      | TCP b/w (Mbps) | RTT (ms) |
    | ------------------------- | -------------- | -------- |
    | `t3.medium` - `t3.medium` | 4520               | 0.202    |
    | `m5.large` - `m5.large`   | 4960               | 0.294         |
    | `c5n.large` - `c5n.large` | 4950               | 0.162         |
    | `t3.medium` - `c5n.large` | 4680               | 0.649         |
    | `m5.large` - `c5n.large`  | 4950               | 0.618         |
    | `m5.large` - `t3.medium`  | 4650               | 0.277         |

    > Region: US East (N. Virginia). Use `Ubuntu Server 22.04 LTS (HVM)` as AMI. Note: Use private IP address when using iPerf within the same region. You'll need iPerf for measuring TCP bandwidth and Ping for measuring Round-Trip time.

    - **TCP Bandwidth and RTT for Same-Type Instances**
      - m5.large - m5.large and c5n.large - c5n.large have the highest bandwidth, reaching nearly 5 Gbps, while t3.medium - t3.medium is slightly lower at 4.52 Gbps.
      - c5n.large - c5n.large has the lowest RTT (0.162 ms), indicating that Compute Optimized instances may have a more stable network connection with lower latency.
  
    - **Network Performance Across Different Instance Types**
      - t3.medium - c5n.large and m5.large - c5n.large have significantly higher RTT (0.649 ms and 0.618 ms, respectively), suggesting that AWS's internal network may have more complex routing or QoS limitations when connecting different instance types.
      - m5.large - t3.medium has a lower RTT (0.277 ms) compared to m5.large - c5n.large (0.618 ms), possibly because t3.medium and m5.large share a similar general-purpose network configuration.

2. (1 mark) What about the network performance for instances deployed in different regions? In order to answer this question, you need to complete the following table.

    | Connection                | TCP b/w (Mbps) | RTT (ms) |
    | ------------------------- | -------------- | -------- |
    | N. Virginia - Oregon      | 381               | 59.562         |
    | N. Virginia - N. Virginia | 4770               | 0.315         |
    | Oregon - Oregon           | 4770               | 0.219         |
 
    > Region: US East (N. Virginia), US West (Oregon). Use `Ubuntu Server 22.04 LTS (HVM)` as AMI. All instances are `c5.large`. Note: Use public IP address when using iPerf within the same region.

    - **Same Region (N. Virginia - N. Virginia) and (Oregon - Oregon)**
      - Bandwidth reaches 4.77 Gbps, and RTT is below 0.32 ms, indicating that AWS’s internal network within the same region is extremely fast, close to theoretical maximum performance.
    
    - **Cross-Region (N. Virginia - Oregon)**
      - Bandwidth drops significantly to 381 Mbps, much lower than the 4.77 Gbps observed within the same region.
      - RTT increases to 59.562 ms, likely due to intercontinental data transmission, possibly traversing multiple network exchange points (Internet Backbone).
      - This suggests that AWS network performance is limited across different regions, potentially affected by geographical distance, AWS internal network policies, and ISP interconnections.
