npm安装windows-build-tools时卡在Successfully installed Python 2.7

#### 原因
windows-build-tools安装脚本bug导致
>“Windows-build-tools脚本存在问题，无法找到VS构建工具安装程序的日志文件。似乎VS构建工具安装程序创建的日志文件已更改。该脚本正在文件夹％USERPROFILE％\ AppData \ Local \ Temp中寻找名称以dd_client_开头的日志文件，但是VS构建工具安装程序似乎并未创建任何以dd_client_开头的文件。”

#### 解决方案
1. 运行`npm install -g windows-build-tools`
2. 在`%temp%`文件夹中找到最新的文件名类似于                   `dd_installer_20210421124746.log`的文件
3. 查看此文件，确保日志中输出了`Closing the installer with exit code 0`
4. 确保你安装了vscode
ps:其实可以直接跳过2–4步，因为你的python环境早就安装好了，重复的步骤安装程序早就执行完毕了
5. 在`%temp%`目录下创建一个名为`dd_client_.log`的文件
6. 编辑5中创建的文件，加入一行`Closing installer. Return code: 3010.`然后保存。

注：`%temp%`在当前用户目录下`%USERPROFILE%\AppData\Local\Temp`，也可以直接在资源管理器中粘贴`%temp%`即可打开你的Windows temp目录。


#### 参考
https://github.com/felixrieseberg/windows-build-tools/issues/244#issuecomment-824213136
https://blog.csdn.net/oqzuser1234asd/article/details/116169889
