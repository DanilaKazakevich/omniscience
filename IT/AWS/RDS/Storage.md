
https://www.reddit.com/r/aws/comments/13qbf7d/ebs_throughput_compared_to_max_io_size_and_iops/\

Block size is a defined chunk of storage where an amount of data can be written or read.

Storage vendors define those block size to be 512 Bytes for HDD & 4K for SSD based.

[](https://stackoverflow.com/posts/61795555/timeline)

Let's try to explain what Throughput and I/O are.

- I/O is the number of accesses to the disk. Each time you need to read a file, you need "at least" to access the file once. However the content is read in "chunks". Each time you read a "chunk", a new I/O is requested. Imagine biting a chocolate bar. You need to access the bar at least once, and you start biting (I/O) until you finish it. Each bite is an I/O. You need several I/Os to swallow the whole bar.
    
- IOPS is I/O per second. Speed. So basically how fast we can perform each bite of the chocolate bar. An IOPS EBS is a volume specialized in performing fast biting: Ñam-Ñam-Ñam vs Ñam------Ñam-----Ñam
    
- Throughput is the amount of info you read in each I/O. Continuing with the example, you can eat the whole chocolate bar in two different ways: small bites (small throughput) or big bites (big throughput), depending on your mouth size. Throughput EBS volume is specialized in performing big biting: Ñam vs Ñaaaaaaaam
    

Are I/O and Throughput related? Sure. If you have to read a big file from EBS, and your throughput is small (aka, your mouth is small so your bites are small) then you need to access (I/O) more frequently until the file is completely read. Ñam-Ñam-Ñam-Ñam

On the other hand, if you have a big mouth (big throughput), then you will need less bites and less I/O. Ñaaaam---Ñaaaam

So in some ways they could balance each other, but....there are corner cases:

a) Imagine you have a 1 really tiny tiny tiny small file (or chocolate nano-bar). --- In this case, even the smallest mouth is enough. With a big or a small mouth you are able to eat the whole nano-bar with just 1 bite.

b) Imagine you have a bucket of zillions of tiny tiny small files ( or chocolate nano-bars) --- In this case, even the smallest mouth is enough to swallow each bar. Big or small throughput won't give you a better performance. However having an IOPS (I/O per second) will boost your performance. A Throughput EBS Volume will perform much much worse than an IOPS Volume.

c) Imagine you have a bucket of zillions of big files. --- So you need Throughput for big files and you need IOPS for multiple access. Then probably you should go towards a EBS General Purpose (it has bursting)

With that you should be able to craft an answer, but for me:

But what about frequent operations with high I/O size? --> EBS General Purpose. Here the "high" and "frequent" ask for a balanced volume.

Infrequent operations with high I/O size? --> EBS Throughput. You need the biggest mouth possible.

Infrequent operations with small I/O size?--> Warning! What is "small" size for you? If they are small for real, then I would probably go towards IOPS becase a big/small mouth (Throughput) will not make a big difference. And in case those "not frequent" becoming a "frequent" (more users?more complexity?) will benefit from the IOPS. Probably you can then also survive with an EBS General Purpose. However, second warning, what does "not frequent operations" mean? Do you mean those files are not frequently accessed? In such case, you should check for a Cold HDD

As always, recommendations, are just recomendations...and the best (because you can get surprised about your sense of "small") is to test performance in cases where you have doubts.

Use cases:

- Work loads -> usually General Purpose Volume
- Databases -> usually IOPS (small data but frequently retrieved)
- Big Data / Data warehouses -> usually Throughput ( big data files)
- Cold HDD -> Cold File Servers (lowest IOPS before moving to Magnetic)


https://www.liainfraservices.com/blog/aws-ebs-disk-throughput-iops/




https://ahmedahamid.com/which-is-better/

Please note that gp3 volume size, IOPs and throughput still impact each other. The maximum throughput that can be reached = (maximum IOPs) x (average IO size). Also, the maximum IOPs = (500) x (provisioned GiB). So, 32 GiB or smaller volumes can't reach the 16,000 IOPs limit.

При дефолтных значения дисков (IOPS, throughput)

The graph shows that gp3 provides a higher consistent IOPS for volumes smaller than 800 GiB. But gp2 provides much higher performance for bigger volumes. Except for small volumes, gp2 allows a much higher throughput.

https://pearlhealth.com/blog/our-technology/how-we-doubled-amazon-rds-throughput-to-support-10x-data-volume-for-less-than-1-day/
не понятно что за бёрст выше чем 3000 IOPS лол
https://docs.aws.amazon.com/ebs/latest/userguide/ebs-io-characteristics.html


https://docs.aws.amazon.com/ebs/latest/userguide/ebs-performance.html
