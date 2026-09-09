# SPDK Integration

Given the NVMe SSDs on the husky server, it is potentially possible to bypass the kernel cache entirely via SPDK. SPDK has a [NVMe driver](https://spdk.io/doc/nvme.html) that provides zero copy data transfer to and from NVMe SSDs that can potentially be integrated into the memcached extstore source code. This allows us to do kernel bypass on the linux filesystem, allowing us to directly measure the latency overhead of the SSD in source code.

There are a few issues with the integration that needs to be kept in mind:

1. SPDK is lockless, while the original source code uses locks for items
2. SPDK uses polling instead of interrupts, so whichever thread owns a queue pair must poll the SPDK completion queue to check for completions rather than blocking and waking up
3. SPDK requires I/O buffers to come from DPDK hugepages, which is different than malloc, so any buffer needs to be rewritten to be allocated with SPDK
4. Once SPDK claims the NVMe SSD, it is no longer visible to the kernel, so it needs its own namespace

This would likely require a substantial rewrite of extstore, and would not reflect the native performance of memcached, but this experiment would be interesting to measure the overhead of SSD vs Network latency.
