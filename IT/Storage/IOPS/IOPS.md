
That is perfectly normal for random I/O performance on a 5400 rpm disk. A 5400 rpm disk can manage about 90 IOPS because the required sector will only go under the head 90 times per second (5400 times per minute).

So with 4KB blocks, that is 4KB * 90 = 360KB/s.

This is broadly in line with what you are seeing.

"In-out operations per second" in a random pattern. Basically take a bunch of small files, scatter them across the storage and see how many the drive can read and write per second.

The higher the iops the better the performance for OS drives/game drives where the OS asks for little files all the time and the drive has to go and find them as fast as possible


клевый тред на реддите 
https://www.reddit.com/r/DataHoarder/comments/y43pp1/what_are_the_theoretical_readwrite_speeds_of_such/

You can't just do Gbps / 8 to get the throughput, you have encoding overhead (8b/10b) for USB 3.0 and 128/132 for USB 3.1+, additionally there is protocol overhead of ~ 10% (and much more on USB 2.0 because of it's half-duplex, polling and general simplicity). For example USB 3.0 is not 750 MBps, but ~ 450MBps; about 100MBps less than Sata 3. You cite USB 3.2 2x2 20Gbps, but that doesn't really exist in any scale in the real market, only some high end desktop motherboards to have some feature checkbox. The enclosures are also very rare and pricey. And 20Gbps USB can not be used full speed on Thunderbolt or USB4 hosts, as all existing chipsets only implement 10Gbps USB fallback and not the 2x2 mode. And with the obnoxiousness I meant that USB naming thing you did. It doesn't add anything of value to your post.

https://www.ixbt.com/data/usb3-gen1-gen2-gen2x2-ext-ssd-test.html