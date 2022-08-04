# Ubuntu永久挂载



## 1 查看分区

查看需要挂载的分区信息

```shell
sudo blkid
```

其中我们需要（名称、UUID、TYPE）

```shell
/dev/sda2: LABEL="data" UUID="46969C8A969C7BDD" TYPE="ntfs" PARTLABEL="Basic data partition" PARTUUID="c59a92ee-b534-4222-8b59-23eec4d7cf75"
```



## 2 修改文件

编辑 `/etc/fstab`

```shell
sudo vim /etc/fstab
```

在最后面添加

第一列为挂载 `UUID`，第二列为挂载目录（必须为空目录），第三列为文件系统类型，第四列为参数，第五列 `0` 表示不备份，第六列必须为 `0` 或 `2`（引导分区为`1`）

```shell
UUID=46969C8A969C7BDD   /mnt    ntfs    defaults        0       2
```



## 3 挂载与卸载

```shell
sudo mount -a
df -kh
```

```shell
sudo umount /dev/sda2
df -kh
```

