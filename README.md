# Linux From Scratch build instructions

[version-check.sh](./version-check.sh)

```code
wget https://github.com/cypherbash/lfs/raw/main/version-check.sh && sudo chmod a+x version-check.sh
```

```code
bash version-check.sh
```

```code
sudo dpkg-reconfigure dash && sudo apt-get install build-essential bison texinfo
```

```code
bash version-check.sh
```

```code
export LFS=/mnt/lfs
nano $HOME/.bashrc
```


```code
sudo su - root
export LFS=/mnt/lfs
nano $HOME/.bashrc
```

```code
passwd
```

```code
gdisk /dev/nvme1n1
```

```code
mkfs -v -t ext4 /dev/nvme1n1p1
```

```code
mkdir -pv $LFS && mount -v -t ext4 /dev/nvme1n1p1 $LFS && mkdir -v $LFS/sources && chmod -v a+wt $LFS/sources
```

```code
wget http://lfs.linux-sysadmin.com/lfs/downloads/12.0-systemd/wget-list && wget --input-file=wget-list --continue --directory-prefix=$LFS/sources
```

```code
cd $LFS/sources && wget http://lfs.linux-sysadmin.com/lfs/downloads/12.0-systemd/md5sums
```

```code
pushd $LFS/sources
  md5sum -c md5sums
popd
```

```code
chown root:root $LFS/sources/* && mkdir -pv $LFS/{etc,var} $LFS/usr/{bin,lib,sbin}
```

```code
for i in bin lib sbin; do
  ln -sv usr/$i $LFS/$i
done
```

```code
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
```

```code
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

## 5.3. GCC-13.2.0 - Pass 1

```code
tar -xf gcc-13.2.0.tar.xz && cd gcc-13.2.0
```


```code
tar -xf ../mpfr-4.2.0.tar.xz
mv -v mpfr-4.2.0 mpfr
tar -xf ../gmp-6.3.0.tar.xz
mv -v gmp-6.3.0 gmp
tar -xf ../mpc-1.3.1.tar.gz
mv -v mpc-1.3.1 mpc
```

```code
case $(uname -m) in
  x86_64)
    sed -e '/m64=/s/lib64/lib/' \
        -i.orig gcc/config/i386/t-linux64
 ;;
esac
```


```code
mkdir -v build && cd build
```

```code
../configure --target=$LFS_TGT --prefix=$LFS/tools --with-glibc-version=2.38 --with-sysroot=$LFS --with-newlib --without-headers --enable-default-pie --enable-default-ssp --disable-nls --disable-shared --disable-multilib --disable-threads --disable-libatomic --disable-libgomp --disable-libquadmath --disable-libssp --disable-libvtv --disable-libstdcxx --enable-languages=c,c++
```

```code
make && make install
```

```code
cd ..
cat gcc/limitx.h gcc/glimits.h gcc/limity.h > `dirname $($LFS_TGT-gcc -print-libgcc-file-name)`/include/limits.h
```

```code
cd .. && rm -rf gcc-13.2.0
```

## 5.4. Linux-6.4.12 API Headers


```code
tar -xf linux-6.4.12.tar.xz && cd linux-6.4.12
```

```code
make mrproper
```

```code
make headers
find usr/include -type f ! -name '*.h' -delete
cp -rv usr/include $LFS/usr
```

```code
cd .. && rm -rf linux-6.4.12
```

## 5.5. Glibc-2.38

```code
tar -xf glibc-2.38.tar.xz && cd glibc-2.38
```

```code
case $(uname -m) in
    i?86)   ln -sfv ld-linux.so.2 $LFS/lib/ld-lsb.so.3
    ;;
    x86_64) ln -sfv ../lib/ld-linux-x86-64.so.2 $LFS/lib64
            ln -sfv ../lib/ld-linux-x86-64.so.2 $LFS/lib64/ld-lsb-x86-64.so.3
    ;;
esac
```

```code
patch -Np1 -i ../glibc-2.38-fhs-1.patch
```

```code
mkdir -v build && cd build
```

```code
echo "rootsbindir=/usr/sbin" > configparms
```

```code
../configure --prefix=/usr --host=$LFS_TGT --build=$(../scripts/config.guess) --enable-kernel=4.14 --with-headers=$LFS/usr/include libc_cv_slibdir=/usr/lib
```

```code
make
```

```code
make DESTDIR=$LFS install
```

```code
sed '/RTLDLIST=/s@/usr@@g' -i $LFS/usr/bin/ldd
```

```code
echo 'int main(){}' | $LFS_TGT-gcc -xc -
readelf -l a.out | grep ld-linux
```

```code
rm -v a.out
```

```code
cd .. && rm -rf glibc-2.38
```

## 5.6. Libstdc++ from GCC-13.2.0

```code
tar -xf gcc-13.2.0.tar.xz && cd gcc-13.2.0
```

```code
mkdir -v build && cd build
```

```code
../libstdc++-v3/configure --host=$LFS_TGT --build=$(../config.guess) --prefix=/usr --disable-multilib --disable-nls --disable-libstdcxx-pch --with-gxx-include-dir=/tools/$LFS_TGT/include/c++/13.2.0
```

```code
make
```

```code
make DESTDIR=$LFS install
```

```code
rm -v $LFS/usr/lib/lib{stdc++,stdc++fs,supc++}.la
```

```code
cd ../.. && rm -rf gcc-13.2.0
```

## 6.2. M4-1.4.19

```code
tar -xf m4-1.4.19.tar.xz && cd m4-1.4.19
```

```code
./configure --prefix=/usr --host=$LFS_TGT --build=$(build-aux/config.guess)
```

```code
make
```

```code
make DESTDIR=$LFS install
```

```code
cd .. && rm -rf m4-1.4.19
```

```code

```

```code

```

```code

```

```code

```





[Linux From Scratch Systemd Online Manual](https://www.linuxfromscratch.org/lfs/view/stable-systemd/)

