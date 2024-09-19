# Wireshark

+ 安装

  ```shell
  sudo add-apt-repository ppa:wireshark-dev/stable 
  # 如果 gpg: keyserver receive failed，手动添加recv-key
  gpg --keyserver keyserver.ubuntu.com --recv-key <recv-key in error info>
  sudo apt update
  apt show wireshark
  sudo apt install wireshark
  ```

+ 配置

  选择了允许非sudo用户抓取数据包，但是好像没什么用。

  据说需要添加wireshark用户组、并给用户组分配root权限读取网络数据，然后将当前用户加入用户组。

+ 启动

  由于我本地没有配置wireshark用户组，需要以超级用户身份启动（sudo wireshark），不然看不到网卡等本地接口信息。

+ 抓包过滤器配置
+ 显示过滤器配置

