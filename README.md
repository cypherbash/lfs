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

```code
mkdir -pv $LFS/{etc,var} $LFS/usr/{bin,lib,sbin}

for i in bin lib sbin; do
  ln -sv usr/$i $LFS/$i
done

case $(uname -m) in
  x86_64) mkdir -pv $LFS/lib64 ;;
esac
```

```code
mkdir -pv $LFS/tools
```

```code
groupadd lfs
useradd -s /bin/bash -g lfs -m -k /dev/null lfs
```

```code
passwd lfs
```

```code
chown -v lfs $LFS/{usr{,/*},lib,var,etc,bin,sbin,tools}
case $(uname -m) in
  x86_64) chown -v lfs $LFS/lib64 ;;
esac
```

```code
su - lfs
```

```code
cat > ~/.bash_profile << "EOF"
exec env -i HOME=$HOME TERM=$TERM PS1='\u:\w\$ ' /bin/bash
EOF
```

```code
cat > ~/.bashrc << "EOF"
set +h
umask 022
LFS=/mnt/lfs
LC_ALL=POSIX
LFS_TGT=$(uname -m)-lfs-linux-gnu
PATH=/usr/bin
if [ ! -L /bin ]; then PATH=/bin:$PATH; fi
PATH=$LFS/tools/bin:$PATH
CONFIG_SITE=$LFS/usr/share/config.site
export LFS LC_ALL LFS_TGT PATH CONFIG_SITE
EOF
```

Execute as root user
```code
[ ! -e /etc/bash.bashrc ] || mv -v /etc/bash.bashrc /etc/bash.bashrc.NOUSE
```

```code
export MAKEFLAGS='-j4'
```

```code
cd /mnt/lfs/sources/
```

## 5.2. Binutils-2.41 - Pass 1

```code
tar -xf binutils-2.41.tar.xz && cd binutils-2.41 && mkdir -v build && cd build
```

```code
time { ../configure ../configure --prefix=$LFS/tools --with-sysroot=$LFS --target=$LFS_TGT --disable-nls --enable-gprofng=no --disable-werror && make && make install; }
```

```code
cd ../.. && rm -rf binutils-2.41
```

```code
tar -xf 
```

```code

```

```code

```



[Linux From Scratch Systemd Online Manual](https://www.linuxfromscratch.org/lfs/view/stable-systemd/)

