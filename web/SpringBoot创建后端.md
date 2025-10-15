# SpringBoot创建后端
## 1、创建一个模块
在需要的创建的项目里面新建一个模块按照以下配置进行配置<img width="855" height="793" alt="image" src="https://github.com/user-attachments/assets/3f22321d-14d8-40ee-aaa6-21013b0d58c9" />
之后再选择一些框架
<img width="859" height="795" alt="image" src="https://github.com/user-attachments/assets/f34e050f-20e2-4532-8660-ff87dad400de" />
## 2、安装Maven
可以直接在官网里面下载[Maven](https://maven.apache.org/download.cgi)
<img width="1481" height="258" alt="image" src="https://github.com/user-attachments/assets/5476920e-2e65-4788-bd3f-60e947c7a4ad" />
下载好之后需要修改`D:\Environment\Maven\apache-maven-3.9.9\conf`(地址可能不一样)中的`settins.xml`
<img width="1043" height="765" alt="image" src="https://github.com/user-attachments/assets/a1106477-77ea-4dae-b3a7-70c1665fc11b" />

代码：
```language
<mirror>
  <id>aliyunmaven</id>
  <mirrorOf>*</mirrorOf>
  <name>阿里云公共仓库</name>
  <url>https://maven.aliyun.com/repository/public</url>
</mirror>
```
<img width="1538" height="832" alt="image" src="https://github.com/user-attachments/assets/41029836-8b33-4805-b223-392fc213c07c" />
```<localRepository>D:\Environment\Maven\rely_on</localRepository>```
之后保存
## 3.使用Maven

1.在设置中搜索maven同时修改一些路径，具体操作如下所示
<img width="1228" height="918" alt="image" src="https://github.com/user-attachments/assets/0f0164a7-7bb2-41b8-bc09-2cf5b05e1911" />

2.设置好之后，找到**后端的`pom.xml`文件，右键添加为`Maven项目`**
3.连接数据库--简写：
<img width="589" height="890" alt="image" src="https://github.com/user-attachments/assets/545f4841-a652-4133-8295-e1cdb73e4bf9" />

<img width="999" height="880" alt="image" src="https://github.com/user-attachments/assets/6cc5aa48-e639-44c4-b04e-440dddf1e75f" />

选择对应的数据库
<img width="607" height="911" alt="image" src="https://github.com/user-attachments/assets/dda66e8e-b72f-4255-842d-7a2254d53065" />

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
