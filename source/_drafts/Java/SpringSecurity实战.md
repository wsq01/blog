

# 表结构
基于 RBAC 权限模型，一共有 5 个表。

| 实体         | 表             | 说明 |
| :--: | :--: | :--: |
| SysUser      | sys_user	      | 用户信息 |
| SysRole      | sys_role	      | 角色信息 |
| SysUserRole  | sys_user_role	| 用户和角色关联 |
| SysMenu      | sys_menu	      | 菜单权限 |
| SysRoleMenu  | sys_role_menu	| 角色和菜单关联 |

5 个表的关系比较简单：
* 一个`SysUser`，可以拥有多个`SysRole`，通过`SysUserRole`存储关联。
* 一个`SysRole`，可以拥有多个`SysMenu`，通过`SysRoleMenu`存储关联。

