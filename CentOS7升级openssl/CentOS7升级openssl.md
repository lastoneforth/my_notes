
# 参考链接

<https://blog.csdn.net/T_LOYO/article/details/130880847>

## 下载依赖和openssl安装包

```bash
yum install -y gcc make perl zlib-devel
wget https://www.openssl.org/source/openssl-1.1.1.tar.gz --no-check-certificate
```

## 解压和编译openssl

```bash
tar -zxvf openssl-1.1.1.tar.gz
cd openssl-1.1.1 

./config   --prefix=/opt/openssl-1.1.1

# 如果没有安装gcc需要先安装
sudo yum install gcc

make
make install
```

## 添加环境变量

```bash
vim /etc/profile
# 将下面2行添加到文件末尾
 export PATH=/opt/openssl-1.1.1/bin:$PATH
 export LD_LIBRARY_PATH=/opt/openssl-1.1.1/lib:$LD_LIBRARY_PATH
```

## 设置软连接

```bash
# 将原来的openssl，做备份
sudo mv /usr/bin/openssl     /usr/bin/openssl_20230525bak
sudo mv /usr/lib64/openssl   /usr/lib64/openssl_20230525bak
# 然后将新安装的OpenSSL做软连接到这个路径
ln  -s  /opt/openssl-1.1.1/bin/openssl   /usr/bin/openssl
```

## 检查版本

```bash
openssl version
```

## 解决openssl缺失libssl.so.1.1问题

参考 <https://blog.csdn.net/zzwlyly/article/details/102719486>
<https://blog.csdn.net/yunyi4367/article/details/78070928>

```bash
sudo sh -c "echo '/usr/lib64' >> /etc/ld.so.conf"
```
