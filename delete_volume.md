you'll often face this error `no space left on device` or `insufficient subnets remaining in the pool` in every 1/2 months.

First one is self explanatory, the worker has used all the space. For the second issue, concourse limits 256 containers per worker https://github.com/concourse/concourse/issues/235

To fix this, you need to

- retire the worker (if it is already running some jobs, you can wait to finish them, or cancel them)
- stop concourse-worker process
- unmount the volumes
- remove the worker directory
- start the worker process


retire the worker
-----------------

From the worker machine
```
$ concourse retire-worker --name concourse-worker-1 --tsa-host=<concourse-web ip>:2222 --tsa-public-key=/etc/concourse/tsa_host_key.pub --tsa-worker-private-key=/etc/concourse/worker_key
```

Confirm from your local machine
```
$ fly -t appscode workers
name                     containers  platform  tags  team  state     version
concourse-worker-1       256         linux     none  none  retiring  2.1
concourse-worker-kubedb  14          linux     cncf  none  running   2.1
```

if it is already running some jobs, you can wait to finish them, or cancel running jobs. after few menuites,
```
$ fly -t appscode workers
name                     containers  platform  tags  team  state    version
concourse-worker-kubedb  8           linux     cncf  none  running  2.1
```

stop concourse-worker process
-----------------------------

ssh to the worker machine
```
$ systemctl stop concourse-worker
```

unmount the volumes and remove worker directory
-----------------------------------------------

```
$ df -H
Filesystem      Size  Used Avail Use% Mounted on
udev             16G     0   16G   0% /dev
tmpfs           3.2G  108M  3.1G   4% /run
/dev/sda3       219G   31G  177G  15% /
tmpfs            16G     0   16G   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs            16G     0   16G   0% /sys/fs/cgroup
overlay         219G   31G  177G  15% /var/lib/concourse/volumes/live/fcb6a400-b6bd-4fe5-5c0f-69f92de6267b/volume
overlay         219G   31G  177G  15% /var/lib/concourse/volumes/live/168cd961-169d-4ca5-60ef-82984783760e/volume
overlay         219G   31G  177G  15% /var/lib/concourse/volumes/live/a4d58084-b591-458c-6a3b-10aa801ad2d2/volume
overlay         219G   31G  177G  15% /var/lib/concourse/volumes/live/ce6d9fe4-2249-4744-6432-5229e8ad6cc1/volume
overlay         219G   31G  177G  15% /var/lib/concourse/volumes/live/b8594724-92eb-4edd-6b4a-33d907c09856/volume
overlay         219G   31G  177G  15% /var/lib/concourse/volumes/live/4e827b7c-58e0-4462-6d7e-7b790d41e475/volume
overlay         219G   31G  177G  15% /var/lib/concourse/volumes/live/0ddb8c85-2389-4367-704b-d5ca9ebb4035/volume
overlay         219G   31G  177G  15% /var/lib/concourse/volumes/live/e6d2ed28-776e-4b32-6b3d-09c9b646e20e/volume
overlay         219G   31G  177G  15% /var/lib/concourse/volumes/live/5ea3361b-ee8e-4dd0-46e2-b574285ea2ae/volume
overlay         219G   31G  177G  15% /var/lib/concourse/volumes/live/9b8c4239-ca1d-4f30-6515-ee91a32bd8f6/volume
overlay         219G   31G  177G  15% /var/lib/concourse/volumes/live/4b3a893b-a68c-449b-7f29-d4ed157213d1/volume
overlay         219G   31G  177G  15% /var/lib/concourse/volumes/live/57201c79-4037-41b5-70c3-0633e5b83418/volume
overlay         219G   31G  177G  15% /var/lib/concourse/volumes/live/51b07764-498b-4aa2-74fb-077c3c54b59e/volume
overlay         219G   31G  177G  15% /var/lib/concourse/volumes/live/8057b9c8-c329-42a8-772d-dd6f7dddb7e5/volume
overlay         219G   31G  177G  15% /var/lib/concourse/volumes/live/07726930-e05e-48c6-55fb-f307a80fde6e/volume
tmpfs           3.2G     0  3.2G   0% /run/user/0
```

you can see, it mounted the overlay volumes into the worker directory(`/var/lib/concourse/volumes/................`)

```
$ cd /var/lib/concourse
$ for dir in $(find .); do umount $dir; done
$ cd
$ rm -rf /var/lib/concourse
```

now check
```
$ df -H                                                                                                    
Filesystem      Size  Used Avail Use% Mounted on                                                                 
udev            2.1G     0  2.1G   0% /dev                                                                       
tmpfs           414M   12M  403M   3% /run                                                                       
/dev/vda1        84G  9.6G   74G  12% /                                                                          
tmpfs           2.1G     0  2.1G   0% /dev/shm                                                                   
tmpfs           5.3M     0  5.3M   0% /run/lock                                                                  
tmpfs           2.1G     0  2.1G   0% /sys/fs/cgroup                                                             
/dev/vda15      110M  3.6M  106M   4% /boot/efi                                                                  
tmpfs           414M     0  414M   0% /run/user/0                                                                
overlay          84G  9.6G   74G  12% /var/lib/concourse/volumes/live/a221b6a6-8dd7-48f5-5a9a-5f28739273c2/volume
overlay          84G  9.6G   74G  12% /var/lib/concourse/volumes/live/0792de0c-7229-428d-5455-3afc4ebebdd8/volume
overlay          84G  9.6G   74G  12% /var/lib/concourse/volumes/live/7ebdeb0b-5901-4b7a-6901-a6c2dab93369/volume
overlay          84G  9.6G   74G  12% /var/lib/concourse/volumes/live/246264c0-3207-4236-7773-9af3763fc980/volume
overlay          84G  9.6G   74G  12% /var/lib/concourse/volumes/live/bf7e704f-66ac-41c7-67db-fc3bef4955c0/volume
overlay          84G  9.6G   74G  12% /var/lib/concourse/volumes/live/d6e311ad-ca18-42f3-7ac8-8cb99cdca633/volume
overlay          84G  9.6G   74G  12% /var/lib/concourse/volumes/live/3e577e94-bf95-4399-7818-58fd2d30e56a/volume
overlay          84G  9.6G   74G  12% /var/lib/concourse/volumes/live/1609cf67-75d0-4556-5bea-8700e3e0786e/volume
overlay          84G  9.6G   74G  12% /var/lib/concourse/volumes/live/99fb9929-f5dc-420e-7de0-09414fd39b76/volume
overlay          84G  9.6G   74G  12% /var/lib/concourse/volumes/live/0e3e0fda-584e-431b-434e-64e60d724654/volume
overlay          84G  9.6G   74G  12% /var/lib/concourse/volumes/live/5973fd99-b54a-4855-44fb-dfffa40d292d/volume
overlay          84G  9.6G   74G  12% /var/lib/concourse/volumes/live/8d8b43f2-e70d-4601-71f2-7673704d2c26/volume
overlay          84G  9.6G   74G  12% /var/lib/concourse/volumes/live/cc8e2759-4e6b-4b01-59c0-b6d2d3485563/volume
overlay          84G  9.6G   74G  12% /var/lib/concourse/volumes/live/2a423b83-bbf7-4e6b-588a-8d569371c67f/volume
overlay          84G  9.6G   74G  12% /var/lib/concourse/volumes/live/964a6524-7c90-407a-52d2-e138b406c1aa/volume
overlay          84G  9.6G   74G  12% /var/lib/concourse/volumes/live/c41c444d-7908-4623-492b-1e96f4521530/volume
overlay          84G  9.6G   74G  12% /var/lib/concourse/volumes/live/008e09a6-ab9c-4cb3-6c53-41b13b336e31/volume
overlay          84G  9.6G   74G  12% /var/lib/concourse/volumes/live/5cf557ec-a8a6-421b-4e30-9c4b87f5ccf1/volume
overlay          84G  9.6G   74G  12% /var/lib/concourse/volumes/live/04bc53f0-3b95-4238-4a65-db38a12b6000/volume 
overlay          84G  9.6G   74G  12% /var/lib/concourse/volumes/live/6c1f476e-5e0e-41fd-4522-964a08b23a79/volume
overlay          84G  9.6G   74G  12% /var/lib/concourse/volumes/live/76ef80bc-18c5-424c-63f1-d8d064365a33/volume
-----------------------
-----------------------
-----------------------
```

start the worker process
------------------------
```
$ systemctl start concourse-worker
```

check from your local machine
```
$ fly -t appscode workers
name                     containers  platform  tags  team  state    version
concourse-worker-1       0           linux     none  none  running  2.1
concourse-worker-kubedb  1           linux     cncf  none  running  2.1
```
