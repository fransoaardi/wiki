# mount additional EBS

## intro
this is to mount additional aws EBS(elastic block storage) on currently running EC2 instance and create file system.

## howto

- create additional EBS and mount on running EC2 instance

- check current status
```shell-script
$ lsblk
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda    202:0    0  16G  0 disk
└─xvda1 202:1    0   8G  0 part /
xvdf    202:80   0  50G  0 disk
```

- check filesystem type
```shell-script
$ lsblk -f
NAME    FSTYPE   LABEL           UUID                                 MOUNTPOINT
loop0   squashfs                                                      /snap/core18/1988
loop1   squashfs                                                      /snap/core/10823
loop2   squashfs                                                      /snap/core/10583
loop3   squashfs                                                      /snap/amazon-ssm-agent/2996
loop4   squashfs                                                      /snap/core18/1944
loop5   squashfs                                                      /snap/amazon-ssm-agent/2333
xvda
└─xvda1 ext4     cloudimg-rootfs abcdefgh-425e-44bb-9de8-7d7877d31328 /
```

- when filesystem is `xfs`
  - grow xvda1 from 8G to 16G
  ```shell-script
  $ sudo growpart /dev/xvda 1
  ```

  - expand xfs filesystem
  ```shell-script
  $ sudo yum install xfsprogs
  $ sudo xfs_growfs -d /
  # because xvda1 mount on /
  ```

  - when filesystem is already created
  파일시스템이 구성된 상태
  ```
  $ sudo file -s /dev/xvdf 
  dev/xvdf: SGI XFS filesystem data (blksz 4096, inosz 512, v2 dirs)
  ```

  - mount 
  ```
  $ sudo mount /dev/xvdf ~/path/to/mount
  ```
  
- when filesystem is `ext4`
  - grow xvda1 from 8G to 16G
  ```shell-script
  $ sudo growpart /dev/xvda 1
  ```
  
  - expand ext4 filesystem
  ```shell-script
  $ sudo resize2fs /dev/xvda1
  ```

## how to make mounted EBS automatically mount on instance reboot

- check UUID with `blkid`
```
$ blkid
/dev/xvda1: LABEL="/" UUID="here-comes-uuid-1" TYPE="xfs" PARTLABEL="Linux" PARTUUID="here-comes-part-uuid"
/dev/xvdf: UUID="here-comes-uuid-2" TYPE="xfs"
```

- make mount automatically on reboot : `/etc/fstab`
```
UUID=here-comes-uuid-1     /                             xfs    defaults,noatime  1   1
UUID=here-comes-uuid-2     /home/ec2-user/path/to/mount  xfs    defaults,nofail   0   2
```

- testing umount/mount
```
$ sudo umount ~/path/to/mount
$ sudo mount -a
```

- mount check
```
$ mount
$ df
```


## references
- https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/recognize-expanded-volume-linux.html

> disk storage 가 꽉찼을때 ebs volume increase 하는 과정
- https://aws.amazon.com/ko/premiumsupport/knowledge-center/ebs-volume-size-increase/
