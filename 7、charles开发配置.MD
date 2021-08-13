# Charles for  windows 使用说明

1. [下载](https://www.charlesproxy.com/)
2. 安装根证书 help>SSL Proxying>Install Root Certificate Help
3. 为浏览器安装证书
   1. help>SSL Proxying>Install Root Certificatte on a Mobile Device or Remote Browser
   2. 根据提示设置当前机器系统代理为10.13.148.53:8888（注意每个机器不一样，一般是本地ip）
   3. 浏览器打开chls.pro/ssl，正常会下载“charles-proxy-ssl-proxying-certificate.pem”
   4. 打开浏览器设置>高级>安全>管理证书>导入>选择pem文件然后选择导入的目录为根证书信任机构
4. 菜单Proxy>SSL Proxying Settings>勾选EnableSSL Proxying，添加Location \*:\*  ,\*:443
5. 菜单Proxy>Windows Proxy