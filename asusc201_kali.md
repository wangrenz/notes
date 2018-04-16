### Chromebook asus c201 安装kali linux到SD卡


#### 1 刻录镜像
下载地址：https://docs.kali.org/kali-on-arm/kali-linux-asus-chromebook-flip

刻录：
```bash
xzcat kali-$version-veyron.img.xz | dd of=/dev/mmcblk1 bs=512k
```
#### 2 扩展根目录

开机按Ctrl + U 进入kali 
