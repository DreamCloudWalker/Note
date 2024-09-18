1. 打开终端输入：
   `cd /Library/Google/GoogleSoftwareUpdate`
2. 然后输入以下命令删除更新组件：
   `sudo rm -rf GoogleSoftwareUpdate.bundle`
3. 自行打开谷歌浏览器的设置页面，提示无法自动更新的字眼，成功关闭自动更新



可能在有些Mac上发现在“/Library”这个根目录下没有Google目录，那么其实在“~/Library”这个用户目录下也有一个Google目录。在该目录下执行操作同样可以禁用自动更新。请执行以下命令：

cd ~/Library/Google 

sudo chown root:wheel GoogleSoftwareUpdate

相当于修改了GoogleSoftwareUpdate这个文件夹的拥有者，而不仅仅是修改了权限，使Google对该文件夹没有写入权限。事实证明这种方式是可行的。重启Chrome完成以后通过“帮助->关于Google Chrome”可以查看结果