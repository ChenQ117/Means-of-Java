# SpringMVC—REST风格

## 简介

- 传统风格

  http://localhost/user/getById?id=1

  http://localhost/user/saveUser

- REST风格

  http://localhost/user/1

  http://localhost/user

- 优点

  - 隐藏资源的访问行为，无法通过地址得知对资源是何种操作
  - 书写简化

- 使用行为动作区分对资源进行了何种操作

  | 路径                     | 功能             | 动作               |
  | ------------------------ | ---------------- | ------------------ |
  | http://localhost/users   | 查询全部用户信息 | GET（查询）        |
  | http://localhost/users/1 | 查询指定用户信息 | GET（查询）        |
  | http://localhost/users   | 添加用户信息     | POST（新增、保存） |
  | http://localhost/users   | 修改用户信息     | PUT（修改、更新）  |
  | http://localhost/users/1 | 删除用户信息     | DELETE（删除）     |

- 根据REST风格对资源进行访问称为RESTful