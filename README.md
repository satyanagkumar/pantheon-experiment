# Pantheon Congestion Control Experiment Report

## Overview
This project evaluates and compares three congestion control algorithms—**Cubic**, **BBR**, and **Vegas** using the Pantheon framework and Mahimahi emulation. Two distinct network environments are simulated to assess protocol performance under different conditions of latency and bandwidth.

---

## 1. Environment Setup

### Host System
- **Virtualization**: Oracle VirtualBox
- **Guest OS**: Ubuntu 20.04 LTS
- **Resources**: 8 CPUs, 6 GB RAM, 25 GB disk

### Initial Setup
```bash
sudo apt-get update
sudo apt install python2 python-pip
sudo apt install virtualenv
virtualenv -p python2 pantheon-env
source pantheon-env/bin/activate
```

## 2. Pantheon Installation
```bash
https://github.com/satyanagkumar/pantheon-experiment.git 
cd pantheon
```

#### Install Dependencies
```bash
sudo ./tools/install_deps.sh
git submodule update --init --recursive
```
#### Install Mahimahi
```bash
sudo apt install mahimahi
```
#### Tunnel Setup
```bash
cd third_party/pantheon-tunnel
./autogen.sh
./configure
make
sudo make install
```
### 3. Experiment Initialization
```bash
cd ~/pantheon/src/experiments
python2 setup.py --setup --schemes "cubic bbr vegas"
```
#### Enable IP Forwarding
```bash
sudo sysctl -w net.ipv4.ip_forward=1
```
#### Install LaTeX (for PDF Report)
```bash
sudo apt install texlive-latex-recommended
```

## 4. Test Methodology

###### Algorithms Tested

Cubic: Default Linux CC protocol, aggressive in high BDP networks

BBR: Google’s model-based CC estimating bandwidth and RTT

Vegas: Delay-based CC aiming to prevent congestion proactively



##### Scenarios

50mbps_10ms - Low-latency, high-bandwidth

1mbps_200ms - High-latency, constrained link

---

## 5. Trace File Creation
```bash
mkdir -p /tmp/traces

# 50 Mbps trace
python2 -c "import sys; [sys.stdout.write('1500\n') for _ in range(4167)]" > /tmp/traces/50mbps.trace

# 1 Mbps trace
python2 -c "import sys; [sys.stdout.write('1500\n') for _ in range(83)]" > /tmp/traces/1mbps.trace
```
## 6. Run Experiments
50 Mbps / 10 ms RTT


```bash
python2 src/experiments/test.py local \
  --schemes "cubic bbr vegas" \
  --data-dir src/experiments/data_50mbps_10ms \
  --runtime 60 \
  --uplink-trace /tmp/traces/50mbps.trace \
  --downlink-trace /tmp/traces/50mbps.trace \
  --prepend-mm-cmds "mm-delay 5" \
  --extra-mm-link-args "--uplink-queue=droptail --downlink-queue=droptail \
    --uplink-queue-args=packets=500 --downlink-queue-args=packets=500"
```

1 Mbps / 200 ms RTT
```bash
python2 src/experiments/test.py local \
  --schemes "cubic bbr vegas" \
  --data-dir src/experiments/data_1mbps_200ms \
  --runtime 60 \
  --uplink-trace /tmp/traces/1mbps.trace \
  --downlink-trace /tmp/traces/1mbps.trace \
  --prepend-mm-cmds "mm-delay 100" \
  --extra-mm-link-args "--uplink-queue=droptail --downlink-queue=droptail \
    --uplink-queue-args=packets=500 --downlink-queue-args=packets=500"
```

## 7. Analyze Results
```bash
# For 50 Mbps
python2 src/analysis/analyze.py --data-dir src/experiments/data_50mbps_10ms

# For 1 Mbps
python2 src/analysis/analyze.py --data-dir src/experiments/data_1mbps_200ms
```


## 8. Output Files
Each run generates the following for each scheme:

*_datalink_throughput_run1.png: Throughput over time

*_datalink_delay_run1.png: Delay (one-way) over time

*_stats_run1.log: Avg throughput, 95th percentile delay, loss rate

pantheon_report.pdf: Full visual report of results

---

## 9. Notes

•Ensure Python 2 and Mahimahi are installed and configured

•IP forwarding must remain enabled during test runs

•Run commands as root or with elevated permissions for networking support

•Graphs and logs will be available in src/experiments/folder/

---

## 10. Conclusion
This assignment provided a comprehensive evaluation of three widely used congestion control protocols Cubic, BBR, and Vegas under two contrasting network conditions. Through trace-based emulation with Mahimahi, it became evident that Cubic aggressively utilizes available bandwidth but suffers from high queuing delays and loss, especially in constrained environments. BBR, while model-driven, often overshoots bandwidth estimates, leading to increased loss in high-latency networks. In contrast, Vegas consistently demonstrated the most latency-friendly behavior, maintaining lower delays and loss rates by adapting its sending rate proactively.

overall, Cubic proved aggressive and throughput-driven, Vegas excelled in low-latency and low-loss environments, and BBR attempted balance but struggled under constrained bandwidth. These experiments highlighted the importance of protocol selection based on specific network characteristics, reaffirming the trade-offs between throughput, delay, and stability in congestion control design.

---
