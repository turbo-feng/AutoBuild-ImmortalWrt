##复制必须进入编辑模式修改

---------- files/etc/uci-defaults/99-custom.sh 
137-145 
1# 设置编译作者信息20251201
FILE_PATH="/etc/openwrt_release" 
NEW_DESCRIPTION="by Turbo" 
sed -i "s/DISTRIB_DESCRIPTION='[^']*'/DISTRIB_DESCRIPTION='$NEW_DESCRIPTION'/" "$FILE_PATH"

2# 设置系统主机名20251201 
HOST_PATH="/etc/config/system" 
NEW_HOSTNAME="TurboWRT" 
sed -i "s/option hostname .*/option hostname '$NEW_HOSTNAME'/" "$HOST_PATH"

----------------------x86-64/build24.sh
30-31 #################################### 使用自建仓库 
git clone --depth=1 https://github.com/turbo-feng/imm-store.git /tmp/store-run-repo 
----------------------##25专用，32-36 
替换内容：
git clone --depth=1 https://github.com/turbo-feng/imm-store.git /tmp/store-apk-repo
#拷贝 run/x86-25 下所有 run 文件和apk文件 到 extra-packages 目录
mkdir -p /home/build/immortalwrt/extra-packages 
cp -r /tmp/store-apk-repo/run/x86-25/* /home/build/immortalwrt/extra-packages/


继续24版本    47-102
# ============= imm仓库内的插件==============
# 定义所需安装的包列表 下列插件你都可以自行删减
PACKAGES=""
PACKAGES="$PACKAGES curl"
#磁盘管理和文件管理器多余
PACKAGES="$PACKAGES luci-i18n-diskman-zh-cn"
PACKAGES="$PACKAGES luci-i18n-firewall-zh-cn"
PACKAGES="$PACKAGES luci-theme-argon"
PACKAGES="$PACKAGES luci-app-argon-config"
PACKAGES="$PACKAGES luci-i18n-argon-config-zh-cn"
#24.10
PACKAGES="$PACKAGES luci-i18n-package-manager-zh-cn"
PACKAGES="$PACKAGES luci-i18n-ttyd-zh-cn"
PACKAGES="$PACKAGES openssh-sftp-server"
PACKAGES="$PACKAGES luci-app-openclash"

# 文件管理器
PACKAGES="$PACKAGES luci-i18n-filemanager-zh-cn"
#会多出NAS管理菜单
#PACKAGES="$PACKAGES luci-i18n-samba4-zh-cn" 
#静态文件服务器dufs(推荐) 配合samba实现远程轻nas管理
#PACKAGES="$PACKAGES luci-i18n-dufs-zh-cn" 
#ram释放
PACKAGES="$PACKAGES luci-app-ramfree" PACKAGES="$PACKAGES luci-i18n-ramfree-zh-cn" 
#CF隧道 
PACKAGES="$PACKAGES luci-app-cloudflared" PACKAGES="$PACKAGES luci-i18n-cloudflared-zh-cn"
#动态域名DDNSGO
PACKAGES="$PACKAGES luci-i18n-ddns-go-zh-cn" PACKAGES="$PACKAGES luci-app-ddns-go"
#自动端口映射，在外网访问服务&&内网设备访问外网
PACKAGES="$PACKAGES luci-app-upnp" PACKAGES="$PACKAGES luci-i18n-upnp-zh-cn"
#DNS选择，提高网页速度
PACKAGES="$PACKAGES luci-i18n-smartdns-zh-cn"
#系统信息统计
PACKAGES="$PACKAGES luci-i18n-statistics-zh-cn" PACKAGES="$PACKAGES luci-app-statistics"
#拦截IP
PACKAGES="$PACKAGES luci-i18n-banip-zh-cn" PACKAGES="$PACKAGES luci-app-banip"
#网易云音乐解锁
PACKAGES="$PACKAGES luci-app-unblockneteasemusic"
#KMS服务器激活
PACKAGES="$PACKAGES luci-app-vlmcsd" PACKAGES="$PACKAGES luci-i18n-vlmcsd-zh-cn"

############# 核心包,仓库自带，PVE必装 ###################### 
PACKAGES="$PACKAGES kmod-mlx5-core" 
PACKAGES="$PACKAGES qemu-ga"

======== shell/custom-packages.sh =======
合并imm仓库以外的第三方插件
PACKAGES="$PACKAGES $CUSTOM_PACKAGES"



-------------------------------------------x86-64/imm.config 
#24-25通用 
284 CONFIG_GRUB_TITLE="TurboWrt" 
295 CONFIG_TARGET_KERNEL_PARTSIZE=48 
296 CONFIG_TARGET_ROOTFS_PARTSIZE=512 
2740 CONFIG_PACKAGE_kmod-mlx5-core=y



------------------------------------------- shell/custom-packages.sh
#!/bin/bash
# ============= imm仓库外的第三方插件==============
# ============= 若启用 则打开注释 ================
# ============= 但此文件也可以处理仓库内的软件去留 本质上是做了一个PACKAGES字符串的拼接 ================

# 各位注意 如果你构建的固件是硬路由 此文件的注释要酌情考虑是否打开 因为硬路由的闪存空间有限 若构建出来过大或者构建失败 记得调整本文件的注释
# 考虑到istore商店的集成与否 属于高频操作 故 目前已将集成store的操作放置在 工作流的UI 选项 用户自行勾选 则集成  不勾选则不集成 以减少修改此文件的次数

# 新增Run安装器 用于快速安装makeself打包的run文件 目前和quickfile的nginx配置冲突 请勿同时集成quickfile
#CUSTOM_PACKAGES="$CUSTOM_PACKAGES luci-app-run"
# 首页和网络向导
#CUSTOM_PACKAGES="$CUSTOM_PACKAGES luci-i18n-quickstart-zh-cn"
# 新增非常好用的文件管理器 sbwml/luci-app-quickfile （luci 23版本不支持 勿集成）
#CUSTOM_PACKAGES="$CUSTOM_PACKAGES bash quickfile luci-app-quickfile luci-i18n-quickfile-zh-cn"
# 高级卸载 by YT Vedio Talk
CUSTOM_PACKAGES="$CUSTOM_PACKAGES luci-app-uninstall"
# 极光主题 by github eamonxg
CUSTOM_PACKAGES="$CUSTOM_PACKAGES luci-theme-aurora luci-app-aurora-config luci-i18n-aurora-config-zh-cn"
# 去广告adghome
CUSTOM_PACKAGES="$CUSTOM_PACKAGES luci-app-adguardhome"
# 代理相关
#CUSTOM_PACKAGES="$CUSTOM_PACKAGES luci-app-openvpn-server luci-i18n-openvpn-server-zh-cn"
#CUSTOM_PACKAGES="$CUSTOM_PACKAGES luci-i18n-openvpn-zh-cn"
#CUSTOM_PACKAGES="$CUSTOM_PACKAGES luci-i18n-dae-zh-cn"
# 已更新到daed 1.28.0 版本
#CUSTOM_PACKAGES="$CUSTOM_PACKAGES luci-i18n-daed-zh-cn"
#CUSTOM_PACKAGES="$CUSTOM_PACKAGES xray-core sing-box hysteria luci-i18n-passwall-zh-cn"
# passwall2 已更新到26.5.1
#CUSTOM_PACKAGES="$CUSTOM_PACKAGES xray-core sing-box hysteria kmod-nft-socket kmod-nft-tproxy luci-app-passwall2 luci-i18n-passwall2-zh-cn"
#CUSTOM_PACKAGES="$CUSTOM_PACKAGES luci-app-openclash luci-compat kmod-tun kmod-inet-diag kmod-nft-tproxy bash curl ip-full unzip"
#CUSTOM_PACKAGES="$CUSTOM_PACKAGES luci-i18n-homeproxy-zh-cn"
# 新版ssrp 支持mihomo
#CUSTOM_PACKAGES="$CUSTOM_PACKAGES kmod-nft-tproxy kmod-nft-socket xray-core naiveproxy luci-app-ssr-plus luci-i18n-ssr-plus-zh-cn"
#CUSTOM_PACKAGES="$CUSTOM_PACKAGES luci-i18n-nikki-zh-cn"
#CUSTOM_PACKAGES="$CUSTOM_PACKAGES luci-app-nekobox"
#CUSTOM_PACKAGES="$CUSTOM_PACKAGES momo luci-app-momo luci-i18n-momo-zh-cn"
# 新增 clashoo by kenzok8 注意若集成clashoo 则不能集成nikki 目前它们俩配置冲突
#CUSTOM_PACKAGES="$CUSTOM_PACKAGES clashoo luci-app-clashoo luci-i18n-clashoo-zh-cn"
# VPN
#CUSTOM_PACKAGES="$CUSTOM_PACKAGES luci-proto-wireguard"
#CUSTOM_PACKAGES="$CUSTOM_PACKAGES luci-app-tailscale-community luci-i18n-tailscale-community-zh-cn"
# 分区扩容 by sirpdboy 
CUSTOM_PACKAGES="$CUSTOM_PACKAGES luci-app-partexp luci-i18n-partexp-zh-cn"
# 看门狗watchdog by sirpdboy
CUSTOM_PACKAGES="$CUSTOM_PACKAGES luci-app-watchdog luci-i18n-watchdog-zh-cn"
# 酷猫主题 by sirpdboy 
CUSTOM_PACKAGES="$CUSTOM_PACKAGES luci-theme-kucat"
# 进阶设置 by sirpdboy 
# 当luci-app-advancedplus插件开启时 需排除冲突项 luci-app-argon-config和luci-i18n-argon-config-zh-cn 减号代表排除
# 若要集成此插件请注意相关issue:https://github.com/wukongdaily/ImmortalWrt-ImageBuilder/issues/521
CUSTOM_PACKAGES="$CUSTOM_PACKAGES luci-app-advancedplus luci-i18n-advancedplus-zh-cn -luci-app-argon-config -luci-i18n-argon-config-zh-cn"
# MosDNS
CUSTOM_PACKAGES="$CUSTOM_PACKAGES luci-app-mosdns luci-i18n-mosdns-zh-cn"
# Turbo ACC 网络加速
CUSTOM_PACKAGES="$CUSTOM_PACKAGES luci-app-turboacc"
# 应用过滤 openappfilter.com
CUSTOM_PACKAGES="$CUSTOM_PACKAGES luci-app-appfilter luci-i18n-appfilter-zh-cn"
# Lucky大吉 
CUSTOM_PACKAGES="$CUSTOM_PACKAGES luci-app-lucky lucky"
# 集客AC
#CUSTOM_PACKAGES="$CUSTOM_PACKAGES luci-app-gecoosac gecoosac"
# 任务设置
CUSTOM_PACKAGES="$CUSTOM_PACKAGES luci-app-taskplan luci-i18n-taskplan-zh-cn"
# Easytier
CUSTOM_PACKAGES="$CUSTOM_PACKAGES easytier luci-app-easytier"
# 统一文件共享
#CUSTOM_PACKAGES="$CUSTOM_PACKAGES webdav2 luci-app-unishare"
# IPSec VPN 服务器
#CUSTOM_PACKAGES="$CUSTOM_PACKAGES luci-i18n-ipsec-vpnd-zh-cn"
# Bandix流量监控 by timsaya
CUSTOM_PACKAGES="$CUSTOM_PACKAGES luci-app-bandix luci-i18n-bandix-zh-cn"
# IPTV 流媒体转发服务器 - rtp2httpd by stackia
#CUSTOM_PACKAGES="$CUSTOM_PACKAGES luci-app-rtp2httpd luci-i18n-rtp2httpd-zh-cn"
# 静态文件服务器dufs
#CUSTOM_PACKAGES="$CUSTOM_PACKAGES luci-i18n-dufs-zh-cn"
#网络测速 by sirpdboy
CUSTOM_PACKAGES="$CUSTOM_PACKAGES luci-app-netspeedtest luci-i18n-netspeedtest-zh-cn"
#皎月连
CUSTOM_PACKAGES="$CUSTOM_PACKAGES luci-app-natpierce"

皎月连
CUSTOM_PACKAGES="$CUSTOM_PACKAGES luci-app-natpierce"
