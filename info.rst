INFO
======

返回节点信息和统计数据。

::

    disque> INFO
    # Server
    disque_version:0.0.1
    disque_git_sha1:0bbd831e
    disque_git_dirty:0
    disque_build_id:cd264e0f87773cd5
    os:Darwin 13.3.0 x86_64
    arch_bits:64
    multiplexing_api:kqueue
    gcc_version:4.2.1
    process_id:528
    run_id:4b4aff36750579760e62d213a311ff7202560caa
    tcp_port:7711
    uptime_in_seconds:374
    uptime_in_days:0
    hz:10
    config_file:

    # Clients
    connected_clients:1
    client_longest_output_list:0
    client_biggest_input_buf:0
    blocked_clients:0

    # Memory
    used_memory:960160
    used_memory_human:937.66K
    used_memory_rss:1519616
    used_memory_peak:960864
    used_memory_peak_human:938.34K
    mem_fragmentation_ratio:1.58
    mem_allocator:libc

    # Jobs
    registered_jobs:0

    # Queues
    registered_queues:1

    # Persistence
    loading:0
    aof_enabled:0
    aof_rewrite_in_progress:0
    aof_rewrite_scheduled:0
    aof_last_rewrite_time_sec:-1
    aof_current_rewrite_time_sec:-1
    aof_last_bgrewrite_status:ok
    aof_last_write_status:ok

    # Stats
    total_connections_received:1
    total_commands_processed:9
    instantaneous_ops_per_sec:0
    total_net_input_bytes:780
    total_net_output_bytes:415
    instantaneous_input_kbps:0.00
    instantaneous_output_kbps:0.00
    rejected_connections:0
    latest_fork_usec:0

    # CPU
    used_cpu_sys:1.54
    used_cpu_user:0.73
    used_cpu_sys_children:0.00
    used_cpu_user_children:0.00


