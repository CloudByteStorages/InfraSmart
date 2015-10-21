# ixl 40G bad performance

#### intel xl

I'm running a few simple tests on -CURRENT with a pair of dual-port Intel XL710 boards,
I have two identical machines connected with patch cables (no switch). 

#### run iperf 

iperf -c 10.0.1.2
------------------------------------------------------------
Client connecting to 10.0.1.2, TCP port 5001
TCP window size: 32.5 KByte (default)
------------------------------------------------------------
[  3] local 10.0.1.1 port 19238 connected with 10.0.1.2 port 5001
[ ID] Interval       Transfer     Bandwidth
[  3]  0.0-10.0 sec  3.91 GBytes  3.36 Gbits/sec

performance is bad

#### run flood ping latency:

# sudo ping -f 10.0.1.2
PING 10.0.1.2 (10.0.1.2): 56 data bytes
.^C
--- 10.0.1.2 ping statistics ---
41927 packets transmitted, 41926 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 0.084/0.116/0.145/0.002 ms

Any ideas on what's going on here? 
Testing 10G ix interfaces between the same two machines results in 9.39 Gbits/sec and flood ping latencies of 17 usec.

#### env, verify

- c states and clock speed - make sure you never go below C1, and fix the clock speed to max. 
  - Sure these parameters also affect the 10G card, but there may be strange interaction that trigger
the power saving modes in different ways.
  - powerd_flags="-a max -b max -n max" in rc.conf
  - safest bet - disable them in bios
  - To disable C-States:
  - sysctl dev.cpu.0.cx_lowest=C1
  - Actually, you want to set hw.acpi.cpu.cx_lowest=C1 instead.  Otherwise you've only changed cpu.0;

- interrupt moderation (may affect ping latency)
  - ixl(4) describes below sysctls, and they default to off:
  - hw.ixl.dynamic_tx_itr: 0
  - hw.ixl.dynamic_rx_itr: 0
  - hw.ixl.rx_itr  - the RX interrupt rate value, set to 8K by default.
  - hw.ixl.tx_itr  - the TX interrupt rate value, set to 4K by default.
  - raise to 20-50k and see what you get in terms of ping latency.
  - Note that 4k on tx means you get to reclaim buffers in the tx queue (unless it is done opportunistically)
every 250us which is dangerously close to the 300us capacity of the queue itself.

- number of queues (32 is a lot i wouldn't use more than 4-8),may affect cpu-socket affinity
  - hw.ixl.max_queues=4 in loader.conf

- tso and flow director - i have seen bad effects of accelerations 
  - so i would run the iperf test with of these features disabled on both sides, and then enable them one at a time. 
  - flow director support is incomplete on FreeBSD
  - ifconfig -tso4 -tso6 -rxcsum -txcsum -lro

- queue sizes 
  - the driver seems to use 1024 slots which is about 1.5 MB queued, 
  - which in turn means you have 300us
  - (and possibly half of that) to drain the queue at 40Gbit/s. 
  - "hw.ixl.ringsz=256" in loader.conf.

- One thing I noticed in top is that one queue irq is using quite a bit of CPU when I run iperf:

   11      0   -92    -     0K  1152K CPU2    2   0:19  50.98% intr{irq293: ixl1:q2}
   11      0   -92    -     0K  1152K WAIT    3   0:02   5.18% intr{irq294: ixl1:q3}
    0      0   -92    0     0K  8944K -      25   0:01   1.07% kernel{ixl1 que}
   11      0   -92    -     0K  1152K WAIT    1   0:01   0.00% intr{irq292: ixl1:q1}
   11      0   -92    -     0K  1152K WAIT    0   0:00   0.00% intr{irq291: ixl1:q0}
    0      0   -92    0     0K  8944K -      22   0:00   0.00% kernel{ixl1 adminq}
    0      0   -92    0     0K  8944K -      31   0:00   0.00% kernel{ixl1 que}
    0      0   -92    0     0K  8944K -      31   0:00   0.00% kernel{ixl1 que}
    0      0   -92    0     0K  8944K -      31   0:00   0.00% kernel{ixl1 que}
   11      0   -92    -     0K  1152K WAIT   -1   0:00   0.00% intr{irq290: ixl1:aq}

With 10G ix interfaces and a throughput of ~9Gb/s, the CPU load is much lower:

   11      0   -92    -     0K  1152K WAIT    0   0:05   7.67% intr{irq274: ix0:que }
    0      0   -92    0     0K  8944K -      27   0:00   0.29% kernel{ix0 que}
    0      0   -92    0     0K  8944K -      10   0:00   0.00% kernel{ix0 linkq}
   11      0   -92    -     0K  1152K WAIT    1   0:00   0.00% intr{irq275: ix0:que }
   11      0   -92    -     0K  1152K WAIT    3   0:00   0.00% intr{irq277: ix0:que }
   11      0   -92    -     0K  1152K WAIT    2   0:00   0.00% intr{irq276: ix0:que }
   11      0   -92    -     0K  1152K WAIT   18   0:00   0.00% intr{irq278: ix0:link}
    0      0   -92    0     0K  8944K -       0   0:00   0.00% kernel{ix0 que}
    0      0   -92    0     0K  8944K -       0   0:00   0.00% kernel{ix0 que}
    0      0   -92    0     0K  8944K -       0   0:00   0.00% kernel{ix0 que}

#### env
- I've rerun the test with Linux 4.3rc6, where I get 13.1 Gbits/sec throughput and 52 usec flood ping latency. 
- Not great either, but in line with earlier experiments with Mellanox NICs and an untuned Linux system.
