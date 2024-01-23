


# 镜像常用命令
## 搜索镜像
可使用`docker search`命令搜索存放在 Docker Hub 中的镜像。
```bash
docker search [OPTIONS] 镜像名称
```
`OPTIONS`：

| 参数         | 默认值	 | 描述 |
| :--: | :--: | :--: |
| --automated  | false	| 只列出自动构建的镜像 |
| --filter, -f |        | 根据指定条件过滤结果 |
| --limit	     | 25	    | 搜索结果的最大条数 |
| --no-trunc	 | false  | 不截断输出，显示完整的输出 |
| --stars, -s	 | 0      | 只展示Star不低于该数值的结果 |

