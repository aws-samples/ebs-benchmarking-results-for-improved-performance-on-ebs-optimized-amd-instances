**Benchmarking tests to prove faster Amazon EBS-optimized performance on Amazon EC2 M6a,C6a and R6a instances**

On April 03rd 2023, we announced an improvement for Amazon Elastic Block Store (EBS) performance on Amazon EC2 compute-optimized C6a and general purpose M6a instance types as you can read from the [announcement](https://aws.amazon.com/about-aws/whats-new/2023/04/sixth-generation-amazon-ec2-instances-ebs-optimized-instance-performance/) . The same  was announced for R6a instance types on June 29th 2023 as per the [announcement](https://aws.amazon.com/about-aws/whats-new/2023/06/amazon-ec2-r6a-instances-ebs-performance/) 

With the latest enhancements to the Nitro system, we have increased the maximum EBS-optimized IOPS by 60% on the 32xlarge instance size and 50% on all other instance sizes for the C6a, M6a and R6a  instance types. The largest 48xlarge and metal sizes now have a maximum IOPS of 240,000, up from prior 160,000. 

This Article does a benchmarking test on EC2 instances from 3 different instance families c6a, M6a and R6a compared with a previous generation M5a.xlarge instance type using the fio tool. I have chosen an xlarge instance types from these families to confirm if they give the same performance as promised. According to the Documentations this instance size should deliver an improved IOPs upto maximum of 40000 IOPs. 

I created a provisioned IOPs volume for the same to make sure that my volume supports the required IOPs. The steps are given below in brief for the C6a instance type and the results obtained from the same tests for the M6a , R6a and one of the previous generation instance types m5a instance are added below. 

1.	Launch an instance of type  c6a.xlarge via console.

<img width="452" alt="image" src="https://github.com/aws-samples/ebs-benchmarking-results-for-improved-performance-on-ebs-optimized-amd-instances/assets/117374837/c391a92c-85fe-4104-abb5-093bb2d0e6f4">

 
2.	Create an IO2 volume with PIOPS greater than 50000 as we are doing tests on a c6a.xlarge instance type to prove it offers 40000 IOPs. Make sure that it is created in the same AZ as that of the instance. Then attach it to the instance as secondary volume.
 
<img width="452" alt="image" src="https://github.com/aws-samples/ebs-benchmarking-results-for-improved-performance-on-ebs-optimized-amd-instances/assets/117374837/4811a6d2-81b9-4154-8004-a50405a854c6">

3. Then attach it to the instance as secondary volume.

 <img width="452" alt="image" src="https://github.com/aws-samples/ebs-benchmarking-results-for-improved-performance-on-ebs-optimized-amd-instances/assets/117374837/fc8489aa-22ea-4f81-a893-d0f44ff3bc39">


4.Login to the instance via SSH, SSM, or instance Connect and install the Benchmarking tool fio. Since I used an Amazon Linux2 instance used yum utiliy to install the command:

$ sudo install fio

Refer the Man page for fio tool here: [fio] (https://linux.die.net/man/1/fio) 


5.Now issue the below commands to verify that the the secondary block is attached, a File system is created and is mounted on /mnt

```
lsblk
NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
nvme0n1       259:0    0    8G  0 disk 
├─nvme0n1p1   259:1    0    8G  0 part /
├─nvme0n1p127 259:2    0    1M  0 part 
└─nvme0n1p128 259:3    0   10M  0 part 
nvme1n1       259:4    0  100G  0 disk 

[root@ip-172-31-73-126 ~]# mkfs.xfs /dev/nvme1n1 -f
meta-data=/dev/nvme1n1           isize=512    agcount=4, agsize=6553600 blks
         =                       sectsz=4096  attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1
data     =                       bsize=4096   blocks=26214400, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=4096  sunit=1 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[root@ip-172-31-73-126 ~]# mount /dev/nvme1n1 /mnt
[root@ip-172-31-73-126 ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        4.0M     0  4.0M   0% /dev
tmpfs           7.7G     0  7.7G   0% /dev/shm
tmpfs           3.1G  420K  3.1G   1% /run
/dev/nvme0n1p1  8.0G  1.5G  6.5G  19% /
tmpfs           7.7G     0  7.7G   0% /tmp
tmpfs           1.6G     0  1.6G   0% /run/user/1000
/dev/nvme1n1    100G  746M  100G   1% /mnt
```


5. Now run fio on the mounted directory with direct random write operations: 

```
sudo fio —directory=/mnt —ioengine=psync —name fio_test_file —direct=1 —rw=randwrite —bs=16k —size=16K —numjobs=16 —time_based —runtime=180 —group_reporting —norandommap
fio_test_file: (g=0): rw=randwrite, bs=(R) 16.0KiB-16.0KiB, (W) 16.0KiB-16.0KiB, (T) 16.0KiB-16.0KiB, ioengine=psync, iodepth=1
...
fio-3.32
Starting 16 processes
fio_test_file: Laying out IO file (1 file / 0MiB)
fio_test_file: Laying out IO file (1 file / 0MiB)
fio_test_file: Laying out IO file (1 file / 0MiB)
fio_test_file: Laying out IO file (1 file / 0MiB)
fio_test_file: Laying out IO file (1 file / 0MiB)
fio_test_file: Laying out IO file (1 file / 0MiB)
fio_test_file: Laying out IO file (1 file / 0MiB)
fio_test_file: Laying out IO file (1 file / 0MiB)
fio_test_file: Laying out IO file (1 file / 0MiB)
fio_test_file: Laying out IO file (1 file / 0MiB)
fio_test_file: Laying out IO file (1 file / 0MiB)
fio_test_file: Laying out IO file (1 file / 0MiB)
fio_test_file: Laying out IO file (1 file / 0MiB)
fio_test_file: Laying out IO file (1 file / 0MiB)
fio_test_file: Laying out IO file (1 file / 0MiB)
fio_test_file: Laying out IO file (1 file / 0MiB)
Jobs: 16 (f=16): [w(16)][31.1%][w=626MiB/s][w=40.0k IOPS][eta 02m:04s]
Jobs: 16 (f=16): [w(16)][32.2%][w=626MiB/s][w=40.0k IOPS][eta 02m:02s]
Jobs: 16 (f=16): [w(16)][33.3%][w=626MiB/s][w=40.0k IOPS][eta 02m:00s]
Jobs: 16 (f=16): [w(16)][34.1%][w=626MiB/s][w=40.0k IOPS][eta 01m:58s]
Jobs: 16 (f=16): [w(16)][34.4%][w=626MiB/s][w=40.0k IOPS][eta 01m:58s
Jobs: 16 (f=16): [w(16)][35.0%][w=622MiB/s][w=39.8k IOPS][eta 01m:57s]
Jobs: 16 (f=16): [w(16)][100.0%][w=626MiB/s][w=40.0k IOPS][eta 00m:00s]
fio_test_file: (groupid=0, jobs=16): err= 0: pid=5340: Wed Jun 14 06:26:51 2023
**write: IOPS=40.1k**, BW=626MiB/s (656MB/s)(110GiB/180001msec); 0 zone resets
clat (usec): min=196, max=2394, avg=398.38, stdev=60.19
lat (usec): min=196, max=2395, avg=398.59, stdev=60.20
clat percentiles (usec):
| 1.00th=[ 273], 5.00th=[ 326], 10.00th=[ 343], 20.00th=[ 363],
| 30.00th=[ 375], 40.00th=[ 388], 50.00th=[ 396], 60.00th=[ 404],
| 70.00th=[ 412], 80.00th=[ 429], 90.00th=[ 453], 95.00th=[ 478],
| 99.00th=[ 529], 99.50th=[ 570], 99.90th=[ 1123], 99.95th=[ 1254],
| 99.99th=[ 1565]
bw ( KiB/s): min=631360, max=878464, per=100.00%, avg=641412.19, stdev=849.64, samples=5744
iops : min=39460, max=54904, avg=40088.26, stdev=53.10, samples=5744
lat (usec) : 250=0.46%, 500=97.27%, 750=1.95%, 1000=0.15%
lat (msec) : 2=0.17%, 4=0.01%
cpu : usr=0.62%, sys=1.56%, ctx=7212248, majf=0, minf=137
IO depths : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
submit : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
complete : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
issued rwts: total=0,7211815,0,0 short=0,0,0,0 dropped=0,0,0,0
latency : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
WRITE: bw=626MiB/s (656MB/s), 626MiB/s-626MiB/s (656MB/s-656MB/s), io=110GiB (118GB), run=180001-180001msec

Disk stats (read/write):
nvme2n1: ios=0/7207547, merge=0/6, ticks=0/2786410, in_queue=2786410, util=100.00%
```
As you can see the IOPs has gone to 40.1K.

The same steps were followed on a **m6a.xlarge** instance and was able to find similar results in instance as you can see below

```
Starting 16 processes
fio_test_file: Laying out IO file (1 file / 0MiB)
fio_test_file: Laying out IO file (1 file / 0MiB)
fio_test_file: Laying out IO file (1 file / 0MiB)
fio_test_file: Laying out IO file (1 file / 0MiB)
fio_test_file: Laying out IO file (1 file / 0MiB)
fio_test_file: Laying out IO file (1 file / 0MiB)
fio_test_file: Laying out IO file (1 file / 0MiB)
fio_test_file: Laying out IO file (1 file / 0MiB)
fio_test_file: Laying out IO file (1 file / 0MiB)
fio_test_file: Laying out IO file (1 file / 0MiB)
fio_test_file: Laying out IO file (1 file / 0MiB)
fio_test_file: Laying out IO file (1 file / 0MiB)
fio_test_file: Laying out IO file (1 file / 0MiB)
fio_test_file: Laying out IO file (1 file / 0MiB)
fio_test_file: Laying out IO file (1 file / 0MiB)
fio_test_file: Laying out IO file (1 file / 0MiB)
Jobs: 16 (f=16): [w(16)][18.3%][w=625MiB/s][w=40.0k IOPS][eta 02m:27s]
Jobs: 16 (f=16): [w(16)][18.9%][w=625MiB/s][w=40.0k IOPS][eta 02m:26s]
Jobs: 16 (f=16): [w(16)][100.0%][w=625MiB/s][w=40.0k IOPS][eta 00m:00s]
fio_test_file: (groupid=0, jobs=16): err= 0: pid=11196: Wed Jun 14 07:09:40 2023
**write: IOPS=40.0k**, BW=626MiB/s (656MB/s)(110GiB/180001msec); 0 zone resets
clat (usec): min=233, max=2233, avg=398.84, stdev=47.13
lat (usec): min=233, max=2233, avg=399.02, stdev=47.12
clat percentiles (usec):
| 1.00th=[ 293], 5.00th=[ 330], 10.00th=[ 347], 20.00th=[ 367],
| 30.00th=[ 379], 40.00th=[ 392], 50.00th=[ 400], 60.00th=[ 404],
| 70.00th=[ 416], 80.00th=[ 433], 90.00th=[ 453], 95.00th=[ 469],
| 99.00th=[ 519], 99.50th=[ 545], 99.90th=[ 758], 99.95th=[ 848],
| 99.99th=[ 1090]
bw ( KiB/s): min=618720, max=805280, per=100.00%, avg=640900.37, stdev=720.23, samples=5744
iops : min=38670, max=50330, avg=40056.27, stdev=45.01, samples=5744
lat (usec) : 250=0.01%, 500=98.26%, 750=1.62%, 1000=0.09%
lat (msec) : 2=0.02%, 4=0.01%
cpu : usr=0.49%, sys=1.34%, ctx=7207460, majf=0, minf=137
IO depths : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
submit : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
complete : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
issued rwts: total=0,7207376,0,0 short=0,0,0,0 dropped=0,0,0,0
latency : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
WRITE: bw=626MiB/s (656MB/s), 626MiB/s-626MiB/s (656MB/s-656MB/s), io=110GiB (118GB), run=180001-180001msec

Disk stats (read/write):
nvme1n1: ios=0/7203049, merge=0/6, ticks=0/2811165, in_queue=2811164, util=99.99%

```
The same results for a **R6a.xlarge** instance for random write operations is given below 

```
sudo fio --directory=/mnt/ --name fio_test_file --direct=1 --rw=randwrite --bs=16k --size=1G --numjobs=16 --time_based --runtime=180 --group_reporting --norandommap
fio_test_file: (g=0): rw=randwrite, bs=(R) 16.0KiB-16.0KiB, (W) 16.0KiB-16.0KiB, (T) 16.0KiB-16.0KiB, ioengine=psync, iodepth=1
...
fio-3.32
Starting 16 processes
Jobs: 16 (f=16): [w(16)][34.1%][w=625MiB/s][w=40.0k IOPS][eta 01m:58s]
Jobs: 16 (f=16): [w(16)][35.0%][w=625MiB/s][w=40.0k IOPS][eta 01m:57s]
Jobs: 16 (f=16): [w(16)][100.0%][w=626MiB/s][w=40.0k IOPS][eta 00m:00s]
fio_test_file: (groupid=0, jobs=16): err= 0: pid=25747: Mon Jul 31 10:06:55 2023
  **write: IOPS=40.1k**, BW=626MiB/s (656MB/s)(110GiB/180001msec); 0 zone resets
    clat (usec): min=272, max=1587, avg=398.49, stdev=37.56
     lat (usec): min=273, max=1588, avg=398.67, stdev=37.56
    clat percentiles (usec):
     |  1.00th=[  322],  5.00th=[  343], 10.00th=[  355], 20.00th=[  371],
     | 30.00th=[  379], 40.00th=[  388], 50.00th=[  396], 60.00th=[  404],
     | 70.00th=[  416], 80.00th=[  424], 90.00th=[  445], 95.00th=[  461],
     | 99.00th=[  506], 99.50th=[  537], 99.90th=[  627], 99.95th=[  676],
     | 99.99th=[  799]
   bw (  KiB/s): min=635488, max=745152, per=100.00%, avg=641317.26, stdev=587.62, samples=5744
   iops        : min=39718, max=46572, avg=40082.35, stdev=36.73, samples=5744
  lat (usec)   : 500=98.82%, 750=1.16%, 1000=0.02%
  lat (msec)   : 2=0.01%
  cpu          : usr=0.50%, sys=1.36%, ctx=7212160, majf=0, minf=150
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwts: total=0,7212079,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
  WRITE: bw=626MiB/s (656MB/s), 626MiB/s-626MiB/s (656MB/s-656MB/s), io=110GiB (118GB), run=180001-180001msec

Disk stats (read/write):
  nvme1n1: ios=0/7207921, merge=0/0, ticks=0/2809425, in_queue=2809425, util=99.98%

```
These values align with the recent improvements in the performance in these EBS optimized instances link


Did the same benchmarking test on a **m5a.xlarge** instance and the result was that the maximum Write IOPs it could reach was 16k 

```

sudo fio --directory=/mnt --ioengine=psync --name fio_test_file --direct=1 --rw=randwrite --bs=16k --size=1G --numjobs=16 --time_based --runtime=180 --group_reporting --norandommap
fio_test_file: (g=0): rw=randwrite, bs=(R) 16.0KiB-16.0KiB, (W) 16.0KiB-16.0KiB, (T) 16.0KiB-16.0KiB, ioengine=psync, iodepth=1
...
fio-3.32
Starting 16 processes
fio_test_file: Laying out IO file (1 file / 1024MiB)
fio_test_file: Laying out IO file (1 file / 1024MiB)
fio_test_file: Laying out IO file (1 file / 1024MiB)
fio_test_file: Laying out IO file (1 file / 1024MiB)
fio_test_file: Laying out IO file (1 file / 1024MiB)
fio_test_file: Laying out IO file (1 file / 1024MiB)
fio_test_file: Laying out IO file (1 file / 1024MiB)
fio_test_file: Laying out IO file (1 file / 1024MiB)
fio_test_file: Laying out IO file (1 file / 1024MiB)
fio_test_file: Laying out IO file (1 file / 1024MiB)
fio_test_file: Laying out IO file (1 file / 1024MiB)
fio_test_file: Laying out IO file (1 file / 1024MiB)
fio_test_file: Laying out IO file (1 file / 1024MiB)
fio_test_file: Laying out IO file (1 file / 1024MiB)
fio_test_file: Laying out IO file (1 file / 1024MiB)
fio_test_file: Laying out IO file (1 file / 1024MiB)
Jobs: 16 (f=16): [w(16)][100.0%][w=250MiB/s][w=16.0k IOPS][eta 00m:00s]
fio_test_file: (groupid=0, jobs=16): err= 0: pid=27178: Mon Jul 17 11:31:25 2023
  **write: IOPS=16.0k**, BW=250MiB/s (263MB/s)(44.0GiB/180002msec); 0 zone resets
    clat (usec): min=268, max=15796, avg=996.46, stdev=182.05
     lat (usec): min=268, max=15797, avg=996.85, stdev=182.05
    clat percentiles (usec):
     |  1.00th=[  553],  5.00th=[  750], 10.00th=[  857], 20.00th=[  938],
     | 30.00th=[  963], 40.00th=[  979], 50.00th=[  996], 60.00th=[ 1012],
     | 70.00th=[ 1029], 80.00th=[ 1057], 90.00th=[ 1156], 95.00th=[ 1221],
     | 99.00th=[ 1319], 99.50th=[ 1385], 99.90th=[ 1582], 99.95th=[ 2008],
     | 99.99th=[ 8586]
   bw (  KiB/s): min=212672, max=576420, per=100.00%, avg=256663.99, stdev=1318.66, samples=5744
   iops        : min=13292, max=36024, avg=16041.49, stdev=82.41, samples=5744
  lat (usec)   : 500=0.84%, 750=4.03%, 1000=48.69%
  lat (msec)   : 2=46.39%, 4=0.02%, 10=0.02%, 20=0.01%
  cpu          : usr=0.38%, sys=1.40%, ctx=2889910, majf=0, minf=125
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwts: total=0,2885512,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
  WRITE: bw=250MiB/s (263MB/s), 250MiB/s-250MiB/s (263MB/s-263MB/s), io=44.0GiB (47.3GB), run=180002-180002msec

Disk stats (read/write):
  nvme2n2: ios=0/2888503, merge=0/4, ticks=0/2830784, in_queue=2830784, util=100.00%

```

This shows the performance improvement in the The sixth generation of Amazon EC2 instances powered by AMD processors. All new instances in the C6a , R6a and M6a instance families will utilize this performance increase at no additional cost. For existing instances, you can simply stop and start your instances to enable this performance increase.

Please refer the read.txt file for the same results for random read operations on these instance types. 



