# 1. Power of 2

# 2. Latency numbers every programmer should know

| Operation | Time |
|-----------|------|
| L1 cache | 0.5ns | 
| Brach mispredict | 5ns |
| L2 cache | 7ns |
| Mutex lock/unlock | 100ns |
| DRAM | 100ns |
| Compress 1Kb with Zippy | 10,000ns = 10us |
| Send 2Kb over 1Gb network | 20,000ns = 20us |
| Read 1Mb sequentially from memory | 250,000ns = 250us |
| Roundtrip within same datacenter | 500,000ns = 500us |
| Disk seek | 10,000,000ns = 10ms |
| Read 1Mb sequentially from network | 10,000,000ns = 10ms |
| Read 1Mb sequentially from disk | 30,000,000ns = 30ms |
| Send packet CA -&gt; Netherlands -&gt; CA | 150,000,000 ns = 150ns |

# 3. Availability Numbers
| Availability | Downtime Per Day | Downtime Per Year |
|-----------|------|------|
| 99% | 14.4 minuts | 3.65 Days|
| 99.9% | 1.44 minuts | 8.77 Hours|
...

# 4. Estimation
- 24 * 60 * 60 = 86,400 seconds per day.
- Estimate Peek QPS as about of average QPS
- Estimate id as 64 bytes. Media as 1MBytes.

Tips:
- **Rounding and Approximation**
- **Write down assumptions**
- **Label units**
- **Commonly asked**: QPS, peak QPS, storage, cache, servers.
