
```bash
# 安装 husky
npm install husky --save-dev
# 启用 git hooks
npx husky install
```
执行完之后文件根目录会多一个 .husky 文件夹：

在安装后自动启用 git hooks
```
npm set-script prepare "husky install"
```
然后你可以看到 package.json 里增加一个 script
```
// package.json
{
  "scripts": {
    "prepare": "husky install"
  }
}
```
注意一个点：yarn 安装是不支持 prepare 这个生命周期的，需要将 prepare 改成 postinstall。

创建一个 precommit hook
npx husky add .husky/pre-commit "npm run lint"

git add .husky/pre-commit
执行完之后在 .husky 目录下会多一个 pre-commit 的文件，里面的 npm run lint 就是这个 hook 要执行的命令，后续要改也可以直接改这个文件。

这个时候 commit 就会先自动执行 npm run lint 了，然后才会 commit。

但是这样解决了以上的问题，当项目大的时候会遇到一些问题，比如每次 lint 是整个项目的文件，文件太多导致跑的时间过久，另外如果这个 lint 是在项目后期接入的话，可能 lint 命令会报很多错误，全量去改可能会有问题。

lint-stadge 只 lint 改动的
基于上面的痛点，lint-stadge 就出现了，它的解决方案就是只检查本次提交所修改(指 git 暂存区[5]里的东西)的问题，这样每次 lint 量就比较小，而且是符合我们的需求的。

lint-staged 用法如下：

1. 安装
npm install -D lint-staged
2. 修改 package.json 配置
{
   "lint-staged": {
 "src/**/*.js": "npm run lint"
   }
}
3. 设置 precommit 为运行 lint-staged
在完成上面的配置之后，可以手动通过 npx lint-staged 来检查暂存区里面的文件。

手动是永远不会手动的，自动的方法就是将 npx lint-staged   设置到 hook 里。

npx husky add .husky/pre-commit "npx lint-staged"
或者直接去改 .husky 下面 precommit 的文件。

到现在我们的代码检查工作流就完成了。在 git commit 的时候就自动的回去帮我们跑检查脚本，而且还是只针对我们本次提交的代码进行检查。