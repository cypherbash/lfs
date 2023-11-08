# Linux From Scratch build instructions

[version-check.sh](./version-check.sh)

```code
bash version-check.sh
```

```code
wget https://github.com/cypherbash/lfs/raw/main/version-check.sh
sudo chmod a+x version-check.sh
./version-check.sh

sudo dpkg-reconfigure dash
sudo apt-get install build-essential bison texinfo
```

```code
sudo su - root
gdisk /dev/nvme1n1
```

```code
mkfs -v -t ext4 /dev/nvme1n1p1
```

```code
mkdir -pv $LFS
mount -v -t ext4 /dev/nvme1n1p1 $LFS
```

```code
mkdir -v $LFS/sources
```

```code
chmod -v a+wt $LFS/sources
```

```code
wget http://lfs.linux-sysadmin.com/lfs/downloads/12.0-systemd/wget-list
wget --input-file=wget-list --continue --directory-prefix=$LFS/sources
```

```code
wget http://lfs.linux-sysadmin.com/lfs/downloads/12.0-systemd/md5sums
pushd $LFS/sources
  md5sum -c md5sums
popd
```

```code
chown root:root $LFS/sources/*
```

[Linux From Scratch Systemd Online Manual](https://www.linuxfromscratch.org/lfs/view/stable-systemd/)

