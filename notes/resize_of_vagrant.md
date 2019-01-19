## 容量拡張(on WSL)

1. A.vmdk -> B.vdi -> resize -> C.vmdk, A.vmdkからC.vmdkへ入れ替え
2. vagrant up && ssh && df -h
3. `sudo fdisk -l` && パーティション "追加" && reboot

### 1. 40GBに容量拡張 && 差し替えまで

```
$ cd /mnt/c/Users/〇〇/VirtualBox\ VMs/目的の場所
$ VBoxManage.exe clonehd A.vmdk B.vdi --format vdi
$ VBoxManage.exe modifyhd B.vdi --resize $((40 * 1024))
$ VBoxManage.exe clonehd B.vdi C.vmdk --format vmdk
$ VBoxManage.exe showvminfo 〇〇 | grep ".vmdk"
SATA Controller (0, 0): C:\Users\〇〇\VirtualBox VMs\A.vmdk (UUID: ...)
$ VBoxManage.exe storageattach 〇〇 --storagectl "SATA Controller" --port 0 --device 0 --type hdd --medium C.vmdk
$ VBoxManage.exe showvminfo 〇〇 | grep ".vmdk"
SATA Controller (0, 0): C:\Users\〇〇\VirtualBox VMs\C.vmdk (UUID: ...)
```

### 2. vagrant up && ssh && df -h

```
$ cd 目的の場所
$ vagrant up
$ vagrant ssh
$ df -h
```

### 3. `sudo fdisk -l` && パーティション "追加" && reboot

- 40GBにディスクは増えていることを確認
- 83,886,080セクタはある
- 20,764,671セクタまでしか使ってない

```
$ sudo fdisk -l
Disk /dev/sda: 40 GiB, 42949672960 bytes, 83886080 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x24995f9e

Device     Boot    Start      End  Sectors  Size Id Type
/dev/sda1  *        2048 18669567 18667520  8.9G 83 Linux
/dev/sda2       18671614 20764671  2093058 1022M  5 Extended
/dev/sda5       18671616 20764671  2093056 1022M 82 Linux swap / Solaris
```

- `sudo cfdisk /dev/sda`
  - New -> 容量MAX(デフォルトでMAXなのでEnterでよい) -> Write -> Quit
- sda3が新しくできる(今回だと、30GBあるsda3)
- 83,886,079セクタまで使用していることがわかる

```
$ sudo fdisk -l
Disk /dev/sda: 40 GiB, 42949672960 bytes, 83886080 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x24995f9e

Device     Boot    Start      End  Sectors  Size Id Type
/dev/sda1  *        2048 18669567 18667520  8.9G 83 Linux
/dev/sda2       18671614 20764671  2093058 1022M  5 Extended
/dev/sda3       20764672 83886079 63121408 30.1G 83 Linux
/dev/sda5       18671616 20764671  2093056 1022M 82 Linux swap / Solaris

Partition table entries are not in disk order.
```

- sda3をLinux LVMにする

