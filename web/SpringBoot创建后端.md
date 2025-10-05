# SpringBoot创建后端
## 1、创建一个模块
在需要的创建的项目里面新建一个模块按照以下配置进行配置![](https://www.fzpersonalweb.xyz/api/uploads/b5ca9d5a-fafa-49f7-8f6c-a42ab0588a3e_后端配置.png)
之后再选择一些框架
![](https://www.fzpersonalweb.xyz/api/uploads/81f0a17a-71a6-4926-90d2-ef1799283df0_框架.png)
## 2、安装Maven
可以直接在官网里面下载[Maven](https://maven.apache.org/download.cgi)
![](https://www.fzpersonalweb.xyz/api/uploads/3e202f0c-6a83-46d6-9952-6283810faac9_Maven.png)
下载好之后需要修改`D:\Environment\Maven\apache-maven-3.9.9\conf`(地址可能不一样)中的`settins.xml`
![](https://www.fzpersonalweb.xyz/api/uploads/ada6555d-3241-4901-ad0c-58c258a5018e_修改maven.png)
代码：
```language
<mirror>
  <id>aliyunmaven</id>
  <mirrorOf>*</mirrorOf>
  <name>阿里云公共仓库</name>
  <url>https://maven.aliyun.com/repository/public</url>
</mirror>
```
之后保存
## 3.使用Maven

1.在设置中搜索maven同时修改一些路径，具体操作如下所示
![](https://www.fzpersonalweb.xyz/api/uploads/1a441c78-8e19-45a9-8103-3038ed64c1f8_image.png)
2.设置好之后，找到**后端的`pom.xml`文件，右键添加为`Maven项目`**
3.连接数据库--简写：
![](https://www.fzpersonalweb.xyz/api/uploads/641462b1-5093-43a4-8de7-436cea6711c2_image.png)
![](https://www.fzpersonalweb.xyz/api/uploads/c1432aa4-2c5c-4cc1-a91a-2879beb700aa_image.png)
选择对应的数据库
![](https://www.fzpersonalweb.xyz/api/uploads/4919724c-6e84-4938-a4ad-ad8f3ff4e09c_image.png)
最后就是将`application.yml`修改为以下内容（密码和数据库名）
```css
spring:
  datasource:
      url: jdbc:mysql://localhost:3306/xxx 
      #jdbc数据库链接 格式为：jdbc:mysql://数据库服务器地址:端口/数据库名
      username: root
      #数据库账号
      password: root
      #数据库密码
      driver-class-name: com.mysql.cj.jdbc.Driver
      #数据库驱动名
      
#以上是MySQL的驱动配置，更多其他配置大家可在网上查阅

```