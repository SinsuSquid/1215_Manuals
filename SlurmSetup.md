# How to set-up & use a Slurm Cluster

written by SinsuSquid (bgkang)
on the day of 14 August 2023

## What is a "Slurm Cluster"?

[Slurm Workload Manager - Quick Start User Guide](https://slurm.schedmd.com/quickstart.html)

![Slurm components](https://slurm.schedmd.com/arch.gif)

## Then, how should I install it?

[따라하며 하는 Slurm 세팅 & 설명, Ubuntu 18.04](https://ai4nlp.tistory.com/25)

1. munge : key cheking between nodes
   
   ```bash
   groupadd -g 991 munge
   useradd  -m -c "MUNGE Uid 'N' Gid Emporium" -d /var/lib/munge -u 991 -g munge -s /sbin/nologin munge
   apt install munge
   ```
   
   - master node
     
     ```bash
     # key generation
     dd if=/dev/urandom of=/etc/munge/munge.key bs=1c count=4M
     
     chmod a-r /etc/munge/munge.key
     chmod u-w /etc/munge/munge.key
     chmod u+r /etc/munge/munge.key
     
     chown munge:munge /etc/munge/munge.key
     
     munge –n | unmunge
     ```
   
   - key transfer
     
     ```bash
     scp /etc/munge/munge.key stokes2:/etc/munge/
     systemctl enable munge
     /etc/init.d/munge restart
     ```
   
   - calc. node
     
     ```bash
     ls -l /etc/munge/munge.key
     chmod a-r /etc/munge/munge.key
     chmod u-w /etc/munge/munge.key 
     chmod u+r /etc/munge/munge.key
     chown munge:munge /etc/munge/munge.key
     systemctl enable munge
     /etc/init.d/munge restart
     ```
   
   - checking - on master node
     
     ```bash
     munge –n | ssh stokes2 unmunge 
     
     # should look like below...
     
     STATUS:    Success (0)
     ENCODE_HOST:    stokes1 (192.168.0.100)
     ENCODE_TIME:    2023-08-14 14:49:52 +0900 (1691995792)
     DECODE_TIME:    2023-08-14 15:49:53 +0900 (1691995793)
     TTL:            300
     CIPHER:        aes128 (4)
     MAC:            sha256 (5)
     ZIP:            none (0)
     UID:            root (0)
     GID:            root (0)
     LENGTH:        0
     ```

2. slurm
- NOTE : 현재 Ubuntu 22.04 apt repository 에서 지원하는 Slurm버전에는 꼭 필요로 하는 기능이 포함되어있지 않습니다. Slurm 홈페이지에서 다운받아 설치하도록 합니다.

- `/etc/slurm/slurm.conf`
  
  ```
  # slurm.conf file generated by configurator easy.html.
  # Put this file on all nodes of your cluster.
  # See the slurm.conf man page for more information.
  #
  ClusterName=stokes
  SlurmctldHost=stokes1
  # 
  #MailProg=/bin/mail 
  MpiDefault=none
  #MpiParams=ports=#-# 
  ProctrackType=proctrack/cgroup
  ReturnToService=1
  SlurmctldPidFile=/var/run/slurmctld.pid
  #SlurmctldPort=6817 
  SlurmdPidFile=/var/run/slurmd.pid
  #SlurmdPort=6818 
  SlurmdSpoolDir=/var/spool/slurmd
  SlurmUser=slurm
  #SlurmdUser=root 
  StateSaveLocation=/var/spool/slurmctld
  SwitchType=switch/none
  TaskPlugin=task/affinity,task/cgroup
  # 
  # 
  # TIMERS 
  #KillWait=30 
  #MinJobAge=300 
  #SlurmctldTimeout=120 
  #SlurmdTimeout=300
  # 
  # 
  # SCHEDULING 
  SchedulerType=sched/backfill
  SelectType=select/cons_tres
  # 
  # 
  # LOGGING AND ACCOUNTING 
  AccountingStorageType=accounting_storage/none
  #JobAcctGatherFrequency=30 
  JobAcctGatherType=jobacct_gather/cgroup
  #SlurmctldDebug=info 
  SlurmctldLogFile=/var/log/slurmctld.log
  #SlurmdDebug=info 
  SlurmdLogFile=/var/log/slurmd.log
  # 
  # 
  # COMPUTE NODES 
  NodeName=stokes1 Gres=gpu:gtx1080:2 CPUs=12 RealMemory=15795 Sockets=1 CoresPerSocket=6 ThreadsPerCore=2 State=UNKNOWN 
  NodeName=stokes2 Gres=gpu:rtx2080:2 CPUs=12 RealMemory=15812 Sockets=1 CoresPerSocket=6 ThreadsPerCore=2 State=UNKNOWN 
  NodeName=stokes3 Gres=gpu:rtx2080:2 CPUs=12 RealMemory=15812 Sockets=1 CoresPerSocket=6 ThreadsPerCore=2 State=UNKNOWN 
  NodeName=stokes4 Gres=gpu:rtx2080:2 CPUs=12 RealMemory=15812 Sockets=1 CoresPerSocket=6 ThreadsPerCore=2 State=UNKNOWN 
  NodeName=stokes5 Gres=gpu:rtx2080:2 CPUs=12 RealMemory=15812 Sockets=1 CoresPerSocket=6 ThreadsPerCore=2 State=UNKNOWN 
  NodeName=stokes6 Gres=gpu:rtx2080:2 CPUs=12 RealMemory=15812 Sockets=1 CoresPerSocket=6 ThreadsPerCore=2 State=UNKNOWN 
  NodeName=stokes7 Gres=gpu:rtx2080:2 CPUs=12 RealMemory=15812 Sockets=1 CoresPerSocket=6 ThreadsPerCore=2 State=UNKNOWN 
  NodeName=stokes8 Gres=gpu:rtx2080:2 CPUs=12 RealMemory=15812 Sockets=1 CoresPerSocket=6 ThreadsPerCore=2 State=UNKNOWN 
  NodeName=stokes9 Gres=gpu:gtx1080:2 CPUs=12 RealMemory=7763 Sockets=1 CoresPerSocket=6 ThreadsPerCore=2 State=UNKNOWN 
  PartitionName=partition Nodes=ALL Default=YES MaxTime=INFINITE State=UP
  ```

- `/etc/slurm/cgroup.conf`
  
  ```
  ###
  # Slurm cgroup support configuration file.
  ###
  CgroupAutomount=yes
  CgroupMountpoint="/sys/fs/cgroup"
  ConstrainCores=yes
  ConstrainDevices=yes
  ConstrainRAMSpace=yes
  ConstrainSwapSpace=yes
  ```

- `/etc/slurm/gres.conf`
  
  ```
  Nodename=stokes1 Name=gpu Type=gtx1080 File=/dev/nvidia[0-1]
  Nodename=stokes2 Name=gpu Type=rtx2080 File=/dev/nvidia[0-1]
  Nodename=stokes3 Name=gpu Type=rtx2080 File=/dev/nvidia[0-1]
  Nodename=stokes4 Name=gpu Type=rtx2080 File=/dev/nvidia[0-1]
  Nodename=stokes5 Name=gpu Type=rtx2080 File=/dev/nvidia[0-1]
  Nodename=stokes6 Name=gpu Type=rtx2080 File=/dev/nvidia[0-1]
  Nodename=stokes7 Name=gpu Type=rtx2080 File=/dev/nvidia[0-1]
  Nodename=stokes8 Name=gpu Type=rtx2080 File=/dev/nvidia[0-1]
  # Nodename=stokes9 Name=gpu Type=gtx1080 File=/dev/nvidia[0-1]
  ```

- `/etc/slurm/`에 있는 모든 `.conf` 파일을 각 node의 `/etc/slurm`에 복사한 후,
  
  - on master node
    
    ```bash
    mkdir /var/spool/slurmctld
    chown slurm: /var/spool/slurmctld
    chmod 755 /var/spool/slurmctld
    touch /var/log/slurmctld.log
    chown slurm: /var/log/slurmctld.log
    touch /var/log/slurm_jobacct.log /var/log/slurm_jobcomp.log
    chown slurm: /var/log/slurm_jobacct.log /var/log/slurm_jobcomp.log
    systemctl enable slurmctld
    /etc/init.d/slurmctld restart
    ```
  
  - on compute nodes
    
    ```bash
    mkdir /var/spool/slurmd
    chown slurm: /var/spool/slurmd
    chmod 755 /var/spool/slurmd
    touch /var/log/slurmd.log
    chown slurm: /var/log/slurmd.log
    systemctl enable slurmd
    /etc/init.d/slurmd restart
    scontrol show nodes
    sinfo –a
    ```

## OK, what's next?

- slurm script 예시
  
  [Slurm 스케쥴러를 이용한 작업의 제출 및 관리](https://dandyrilla.github.io/2017-04-11/jobsched-slurm/)
  
  ```bash
  ------- sleep.sh -------
  #!/bin/bash
  #
  #SBATCH --job-name=test
  #SBATCH --output=res.txt
  #SBATCH --ntasks=1
  #SBATCH --time=10:00
  
  srun sleep 120 
  srun hostname
  -----------------------
  
  sbatch sleep.sh
  squeue
  ```

- slurm command
  
  | command      | purpose         |
  | ------------ | --------------- |
  | sbatch       | 작업 제출           |
  | scancel PID  | 작업 삭제           |
  | squeue       | 작업상태 확인         |
  | smap         | 작업 및 node 상태 확인 |
  | sinfo        | node 정보 확인      |
  | sview (head) | GUI로 상태 확인      |

### Troubleshooting

- 일정 시간 후 slurm node가 down으로 변할 때
  
  - 방화벽 설정으로 인한 문제입니다.
  - 모든 node에서 6817, 6818번 port를 open합니다. 

- job을 batch하고 바로 중단되며, node가 drain으로 전환되는 경우
  
  - Ubuntu 22.04 버전으로 업데이트되면서 cgroup/v2로 업그레이드되며 발생하는 문제입니다.
  
  - Slurm 22.05 버전 이후로 cgroup/v2를 지원하니 다시 설치하길 바랍니다.
  
  - cgroup/v1과 cgroup/v2의 hybrid를 지원하지 않습니다.
  
  - Stokes의경우 cgroup/v1을 강제로 꺼버린 다음 `/etc/slurm/cgroup.conf`에 `CgroupPlugin=cgroup/v1 (오타아님)`으로 설정해버려서 아예 cgroup init을 안하도록 설정해버렸습니다. (이 경우 그래픽 드라이버 재설치가 필요합니다.)
  
  - `/var/log/slurmctld.log`를 참고해 문제를 확인합니다.
    
    ```
    ...
    [2023-09-06T09:57:38.996] error: slurmd error running JobId=** on node(s)=stokes1: Plugin initialization failed
    [2023-09-06T09:57:38.996] drain_nodes: node stokes1 state set to DRAIN
    ...
    ```
  
  - `/var/log/slurmd.log` on stokes1
    
    ```
    ...
    [2023-09-06T09:57:38.981] task/affinity: task_p_slurmd_batch_request: task_p_slurmd_batch_request: 28
    [2023-09-06T09:57:38.981] task/affinity: batch_bind: job 28 CPU input mask for node: 0x003
    [2023-09-06T09:57:38.981] task/affinity: batch_bind: job 28 CPU final HW mask for node: 0x041
    [2023-09-06T09:57:38.982] Launching batch job 28 for UID 1003
    [2023-09-06T09:57:38.995] [28.batch] error: unable to mount cpuset cgroup namespace: Device or resource busy
    [2023-09-06T09:57:38.995] [28.batch] error: unable to create cpuset cgroup namespace
    [2023-09-06T09:57:38.995] [28.batch] error: unable to mount memory cgroup namespace: Device or resource busy
    [2023-09-06T09:57:38.995] [28.batch] error: unable to create memory cgroup namespace
    [2023-09-06T09:57:38.995] [28.batch] error: failure enabling memory enforcement: Unspecified error
    [2023-09-06T09:57:38.995] [28.batch] error: Couldn't load specified plugin name for task/cgroup: Plugin init() callback failed
    [2023-09-06T09:57:38.995] [28.batch] error: cannot create task context for task/cgroup
    [2023-09-06T09:57:38.995] [28.batch] error: job_manager: exiting abnormally: Plugin initialization failed
    [2023-09-06T09:57:38.995] [28.batch] sending REQUEST_COMPLETE_BATCH_SCRIPT, error:1011 status:0
    [2023-09-06T09:57:38.997] [28.batch] done with job
    ```
