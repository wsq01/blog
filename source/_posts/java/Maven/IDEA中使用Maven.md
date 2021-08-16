


# IDEA 中配置 Maven
将本地安装的 Maven 配置到 IntelliJ IDEA 中操作步骤如下。

1. 在 IntelliJ IDEA 工作区上方的菜单栏中选择`File`，然后在下拉菜单中选则`Settings...`，如下图。

{% asset_img 1.png IDEA Settings %}

2. 在弹出对话框中，展开`Build,Execution,Deployment`，在`Build Tools`下选择 Maven，如下图。

{% asset_img 2.png IDEA 中配置本地 Maven %}

在 IntelliJ IDEA 中配置 Maven 需要进行 3 步：
1. 在 Maven home path 中，指定本地 Maven 的位置；
2. 勾选 User Settings file 后面的 Override，并指定本地仓库的 setting.xml 文件；
3. 勾选 Local repository 后面的 Override，并指定本地仓库的地址。

# 新建 Maven 项目
在 IntelliJ IDEA 中新建 Maven 项目操作步骤如下。
1. 在 IntelliJ IDEA 工作区上方的菜单栏中选择`File`，在下拉菜单中中选中`New`，然后选择`Project`，如下图。

{% asset_img 3.png IDEA 新建Maven 项目1 %}

2. 在左侧的选项中选择 Maven，勾选`Create from archetype`选项，然后在下面选择合适的`Maven Archetype`（模型），最后点击下方的`Next`按钮，如下图。

{% asset_img 4.png IDEA 选择 Maven 原型 %}

注意：此处我们也可以不勾选`Create from archetype`选项，直接点击下方的`Next`按钮，来直接创建一个简单的 Maven 项目。

3. 展开`Artifact Coordinates`，分别输入项目名称（`name`）、存储位置（`Location`）、`GroupId、ArtifactId`以及`Version`等信息，信息输入完成后，点击`Next`按钮。

{% asset_img 5.png IDEA 填写 Maven 项目信息 %}

4. 在该页面可以设置 Maven 的主目录和本地仓库信息，除此之外，我们还可以在下面的属性（`Properties`）列表中，检查和修改项目的信息。配置完成后，点击`Finish`按钮，完成项目的创建。

{% asset_img 6.png IDEA 设置 Maven 主目录和本地仓库 %}

5. 返回 IntelliJ IDEA 工作区，会发现 Maven 项目创建完成，其目录结构如图所示。

{% asset_img 7.png Maven 项目结构 %}

# 导入Maven项目
相比新建 Maven 项目，实际工作中使用更多的是将已有的 Maven 项目导入 IntelliJ IDEA 中，具体步骤如下。
 

1. 在 IntelliJ IDEA 工作区上方的菜单栏中选择`File`，在下拉菜单选中`Open`。

{% asset_img 8.png IDEA 工作区菜单 open %}

2. 选择需要导入的 Maven 项目，如下图。

{% asset_img 9.png IDEA 选择需要导入的项目 %}

3. 若该项目是第一次在 IDEA 中打开，IntelliJ IDEA 会对其进行分析。如果检测到多个配置，则提示用户选择需要使用的配置，最后单击`OK`按钮。

{% asset_img 10.png 选择配置 %}

4. 返回 IDEA 工作区，可以看到项目已经导入。
# 执行Maven命令
在工作区的最右侧，IntelliJ IDEA 为我们提供了一个十分实用的窗口：Maven 工具窗口，通过它我们几乎可以完成所有与 Maven 相关的操作。

{% asset_img 11.png IDEA Maven 工具窗口 %}

在 Maven 工具窗口中，我们可以通过以下 3 种方式中执行 Maven 命令：
* 使用 Run Anything 窗口
* 使用 Maven 工具窗口的上下文菜单
* 为一个或一组 Maven 目标创建运行配置。

## 使用 Run Anything 窗口
在 Maven 工具窗口的工具栏上，点击“m”按钮，或在 IntelliJ IDEA 中连续两次按下`Ctrl`键，即可打开`Run Anything`窗口。

{% asset_img 12.png IDEA Run Anything 窗口 %}

在`Run Anything`窗口输入或选择下方列表中 Maven 目标，回车即可执行，如图所示。

{% asset_img 13.png IDEA Run Anything 执行 Maven 目标 %}

Maven 目标执行结果如图所示。

{% asset_img 14.png IDEA Maven 目标执行结果 %}

如果一个项目包含多个模块，且需要在特定的模块中执行 Maven 目标，则需要在`Run Anything`窗口右上角从“Project”列表中切换到所需的模块，然后再执行的 Maven 目标执行。 

{% asset_img 15.png IDEA Run Anything 窗口切换模块 %}

## 使用上下文菜单
在 Maven 工具窗口，点击项目名下面的`LifeCycle`，如图所示。

{% asset_img 16.png IDEA Maven 工具窗口 Lifecycle %}

右键单击 Maven 目标，在上下文菜单选择 Run '项目名 [Maven 目标]'（例如：`Run 'secondEclipse [clean]'`），即可执行该目标，如图所示。

{% asset_img 17.png 上下文菜单 %}

## 通过运行配置执行 Maven 目标
IntelliJ IDEA 还可以创建运行配置，来执行一个或多个 Maven 目标，步骤如下。

1. 在 Maven 工具窗口，点击项目名下面的`LifeCycle`。

2. 按住`Ctrl`键同时选择一个或多个 Maven 目标，然后单击鼠标右键，在下拉菜单中选择`Create Run Configuration...`或者`Modify Run Configuration... `，如图所示。 

{% asset_img 18.png 为 Maven 目标创建运行配置 %}

3. 在创建运行配置窗口，我们可以为 Maven 目标指定 Maven 命令和参数等，设置完成后，点击 OK 按钮保存该运行配置，如图所示。

{% asset_img 19.png IDEA 配置运行配置 %}

4. 配置完成后，在 Maven 工具窗口下自动生成了一个`Run Configurations`节点，再该节点下可以看到运行配置列表，如图所示。

{% asset_img 20.png IDEA 运行配置列表 %}

5. 在运行配置列表中，双击目标，或右键点击该目标然从上下文菜单中选择`Run`，即可运行该目标。

{% asset_img 21.png 通过运行配置执行目标 %}