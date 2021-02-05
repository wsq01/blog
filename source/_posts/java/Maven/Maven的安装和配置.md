

# 在 windows 上安装 Maven
在安装 Maven 之前要确保安装了 JDK。

访问 Maven 下载页面：`https://maven.apache.org/download.cgi`，下载`apache-maven-3.6.3-bin.zip`。

{% asset_img img1.png %}

将压缩包解压到指定的目录中，比如：`C:\path\apache-maven-3.6.3`。

接着需要设置 Maven 的环境变量。

{% asset_img img2.png %}
{% asset_img img3.png %}

在 cmd 中输入`mvn -v`，检查 Maven 是否安装成功。
# 安装目录分析
Maven 目录结构：

{% asset_img img4.png %}

