---
title: Linux VM へのディスクの追加 | Microsoft Docs
description: Linux VM に永続ディスクを追加する方法について説明します。
keywords: Linux 仮想マシン,リソース ディスクの追加
services: virtual-machines-linux
documentationcenter: ''
author: rickstercdn
manager: timlt
editor: tysonn
tags: azure-resource-manager

ms.service: virtual-machines-linux
ms.topic: article
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.date: 09/06/2016
ms.author: rclaus

---
# Linux VM へのディスクの追加
この記事では、メンテナンスやサイズ変更により VM が再プロビジョニングされる場合でもデータを保持できるように、永続ディスクを VM に接続する方法について説明します。ディスクを追加するには、Resource Manager モードで構成された [Azure CLI](../xplat-cli-install.md) が必要です (`azure config mode arm`)。

## クイック コマンド
以下のコマンド例の &lt; と &gt; で囲まれた値は、実際の環境の値に置き換えてください。

```bash
azure vm disk attach-new <myuniquegroupname> <myuniquevmname> <size-in-GB>
```

## ディスクの接続
新しいディスクには簡単に接続できます。「`azure vm disk attach-new <myuniquegroupname> <myuniquevmname> <size-in-GB>`」と入力して、VM 用に新しい GB ディスクを作成し、接続します。ストレージ アカウントを明示的に特定しない場合、作成するディスクは、OS ディスクと同じストレージ アカウントに配置されます。次のようになります。

```bash
azure vm disk attach-new myuniquegroupname myuniquevmname 5
```

出力

```bash
info:    Executing command vm disk attach-new
+ Looking up the VM "myuniquevmname"
info:    New data disk location: https://cliexxx.blob.core.windows.net/vhds/myuniquevmname-20150526-0xxxxxxx43.vhd
+ Updating VM "myuniquevmname"
info:    vm disk attach-new command OK
```

## Linux VM を接続して新しいディスクをマウントする
> [!NOTE]
> このトピックでは、ユーザー名とパスワードを使用して VM に接続します。公開キーおよび秘密キーのペアを使用して VM と通信する方法については、[Azure 上の Linux における SSH の使用方法](virtual-machines-linux-mac-create-ssh-keys.md)に関するページをご覧ください。`azure vm quick-create` コマンドを使って作成された VM の **SSH** 接続を `azure vm reset-access` コマンドを使って **SSH** アクセスを完全にリセットしたり、ユーザーを追加または削除したりできます。また、アクセスをセキュリティで保護するための公開キー ファイルを追加することもできます。
> 
> 

Linux VM から使用できるように新しいディスクのパーティション分割、フォーマット、マウントを行うには、SSH で Azure VM に接続する必要があります。**ssh** を使用した接続に慣れていない場合は、`ssh <username>@<FQDNofAzureVM> -p <the ssh port>` 形式のコマンドを使用します。コマンドは次のようになります。

```bash
ssh ops@myuni-westu-1432328437727-pip.westus.cloudapp.azure.com -p 22
```

出力

```bash
The authenticity of host 'myuni-westu-1432328437727-pip.westus.cloudapp.azure.com (191.239.51.1)' can't be established.
ECDSA key fingerprint is bx:xx:xx:xx:xx:xx:xx:xx:xx:x:x:x:x:x:x:xx.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'myuni-westu-1432328437727-pip.westus.cloudapp.azure.com,191.239.51.1' (ECDSA) to the list of known hosts.
ops@myuni-westu-1432328437727-pip.westus.cloudapp.azure.com's password:
Welcome to Ubuntu 14.04.2 LTS (GNU/Linux 3.16.0-37-generic x86_64)

* Documentation:  https://help.ubuntu.com/

System information as of Fri May 22 21:02:32 UTC 2015

System load: 0.37              Memory usage: 2%   Processes:       207
Usage of /:  41.4% of 1.94GB   Swap usage:   0%   Users logged in: 0

Graph this data and manage this system at:
  https://landscape.canonical.com/

Get cloud support with Ubuntu Advantage Cloud Guest:
  http://www.ubuntu.com/business/services/cloud

0 packages can be updated.
0 updates are security updates.

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

ops@myuniquevmname:~$
```

これで VM に接続されたので、ディスクを接続する準備ができました。まず、`dmesg | grep SCSI` を使用してディスクを探します (新しいディスクの検出に使用する方法は異なる場合があります)。この場合、次のようになります。

```bash
dmesg | grep SCSI
```

出力

```bash
[    0.294784] SCSI subsystem initialized
[    0.573458] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 252)
[    7.110271] sd 2:0:0:0: [sda] Attached SCSI disk
[    8.079653] sd 3:0:1:0: [sdb] Attached SCSI disk
[ 1828.162306] sd 5:0:0:0: [sdc] Attached SCSI disk
```

このトピックで必要なのは `sdc` ディスクです。使用しているディスクを `sdc` と想定して、`sudo fdisk /dev/sdc` でディスクをパーティション分割します。それをパーティション 1 上のプライマリ ディスクにして、それ以外は既定値をそのまま使用します。

```bash
sudo fdisk /dev/sdc
```

出力

```bash
Device contains neither a valid DOS partition table, nor Sun, SGI or OSF disklabel
Building a new DOS disklabel with disk identifier 0x2a59b123.
Changes will remain in memory only, until you decide to write them.
After that, of course, the previous content won't be recoverable.

Warning: invalid flag 0x0000 of partition table 4 will be corrected by w(rite)

Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-10485759, default 2048):
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-10485759, default 10485759):
Using default value 10485759
```

プロンプトで「`p`」と入力して、パーティションを作成します。

```bash
Command (m for help): p

Disk /dev/sdc: 5368 MB, 5368709120 bytes
255 heads, 63 sectors/track, 652 cylinders, total 10485760 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x2a59b123

   Device Boot      Start         End      Blocks   Id  System
/dev/sdc1            2048    10485759     5241856   83  Linux

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```

**mkfs** コマンドを使用し、ファイル システムの種類とデバイス名を指定して、パーティションにファイル システムを作成します。このトピックでは、前の内容から `ext4` と `/dev/sdc1` を使用しています。

```bash
sudo mkfs -t ext4 /dev/sdc1
```

出力

```bash
mke2fs 1.42.9 (4-Feb-2014)
Discarding device blocks: done
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
327680 inodes, 1310464 blocks
65523 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=1342177280
40 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks:
    32768, 98304, 163840, 229376, 294912, 819200, 884736
Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
```

次に、`mkdir` を使用して、ファイル システムをマウントするディレクトリを作成します。

```bash
sudo mkdir /datadrive
```

`mount` を使用して、ディレクトリをマウントします。

```bash
sudo mount /dev/sdc1 /datadrive
```

これで、データ ディスクを `/datadrive` として使用する準備ができました。

```bash
ls
```

出力

```bash
bin   datadrive  etc   initrd.img  lib64       media  opt   root  sbin  sys  usr  vmlinuz
boot  dev        home  lib         lost+found  mnt    proc  run   srv   tmp  var
```

再起動後にドライブを自動的に再マウントするために、そのドライブを /etc/fstab ファイルに追加する必要があります。また、ドライブを参照する際に、デバイス名 (`/dev/sdc1` など) だけでなく、UUID (汎用一意識別子) を /etc/fstab で使用することもお勧めします。UUID を使用すると、OS が起動中にディスク エラーを検出した場合に、間違ったディスクが特定の場所にマウントされるのを防ぐことができます。その後、残りのデータ ディスクは、その同じデバイス ID に割り当てられます。新しいドライブの UUID を確認するには、**blkid** ユーティリティを使用します。

```bash
sudo -i blkid
```

出力は、次のようになります。

```bash
/dev/sda1: UUID="11111111-1b1b-1c1c-1d1d-1e1e1e1e1e1e" TYPE="ext4"
/dev/sdb1: UUID="22222222-2b2b-2c2c-2d2d-2e2e2e2e2e2e" TYPE="ext4"
/dev/sdc1: UUID="33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e" TYPE="ext4"
```

> [!NOTE]
> **/etc/fstab** ファイルを不適切に編集すると、システムが起動できなくなる可能性があります。編集方法がはっきりわからない場合は、このファイルを適切に編集する方法について、ディストリビューションのドキュメントを参照してください。編集する前に、/etc/fstab ファイルのバックアップを作成することもお勧めします。
> 
> 

次に、テキスト エディターで **/etc/fstab** ファイルを開きます。

```bash
sudo vi /etc/fstab
```

この例では、前の手順で作成した新しい **/dev/sdc1** デバイスに対して UUID 値を使用し、マウント ポイントとして **/datadrive** を使用します。次の行を **/etc/fstab** ファイルの末尾に追加します。

```bash
UUID=33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e   /datadrive   ext4   defaults   1   2
```

> [!NOTE]
> この後、fstab を編集せずにデータ ディスクを削除すると VM は起動できません。ディストリビューションのほとんどに `nofail` または `nobootwait` fstab オプションが用意されています。これにより起動時にディスクのマウントが失敗しても、システムを起動できます。これらのパラメーターの詳細については、使用しているディストリビューションのドキュメントを参照してください。
> 
> 

### Azure における Linux の TRIM/UNMAP サポート
一部の Linux カーネルでは、ディスク上の未使用ブロックを破棄するために TRIM/UNMAP 操作がサポートされます。これは主に、Standard Storage で、削除されたページが無効になり、破棄できるようになったことを Azure に通知するときに役立ちます。これによって、サイズの大きいファイルを作成して削除する場合のコストを節約できます。

Linux VM で TRIM のサポートを有効にする方法は 2 通りあります。通常どおり、ご使用のディストリビューションで推奨される方法をお問い合わせください。

* 次のように、`/etc/fstab` で `discard` マウント オプションを使用します。
  
        UUID=33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e   /datadrive   ext4   defaults,discard   1   2
* または、`fstrim` コマンドを手動でコマンド ラインから実行するか、crontab に追加して定期的に実行することができます。
  
    **Ubuntu**
  
        # sudo apt-get install util-linux
        # sudo fstrim /datadrive
  
    **RHEL/CentOS**
  
        # sudo yum install util-linux
        # sudo fstrim /datadrive

## トラブルシューティング
[!INCLUDE [virtual-machines-linux-lunzero](../../includes/virtual-machines-linux-lunzero.md)]

## 次のステップ
* 新しいディスクは、[fstab](http://en.wikipedia.org/wiki/Fstab) ファイルにその情報を書き込まない限り、再起動しても VM で使用できないことに注意してください。
* [Linux マシンのパフォーマンスの最適化](virtual-machines-linux-optimization.md)に関する推奨事項を読んで、Linux VM が正しく構成されていることを確認します。
* ディスクを追加してストレージ容量を拡張し、[RAID を構成](virtual-machines-linux-configure-raid.md)してパフォーマンスを強化します。

<!---HONumber=AcomDC_0914_2016-->