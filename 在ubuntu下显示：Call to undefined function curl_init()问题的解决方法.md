在ubuntu下显示：Call to undefined function curl_init()问题的解决方法



声明：我使用的是php5.5

查看你的php版本的方法是输入：php  -v

如果显示：The program 'php5' is currently not installed. You can install it by typing:
apt-get install php5-cli

那么先输入：sudo apt-get install php5-cli



好了，进入正题

产生问题原因：你可能是在windows下写了关于curl的一个有用的操作，但是把代码放上你的托管服务器上的时候出现了这个问题。原因很简单，你没有在你的托管服务器上安装curl

解决方法：安装curl即可

具体操作：

1、sudo apt-get update

2、sudo sed -i -e 's/us.archive.ubuntu.com/archive.com/g' /etc/apt/sources.list

3、sudo apt-get install libcurl3 php5-curl