# Requirement 1
# Jenkins
## 人员构成
### 组员
* 徐家辉
* 顾一辉
* 茅悦田
* 原帅
## 需要准备的项
* Java
* Jenkins
* Maven
* Tomcat
* 腾讯云服务器
* 项目代码（本文以 *springboot* 项目为例）
## 搭建CI环境
### 为了实现本地修改代码push到Github上之后Jenkins能实现自动构建，需要在Github个人页面中完成webhook配置和取得Token以便jenkins访问
1. 生成Github个人token并妥善保存 <br></br>
![avatar](https://raw.githubusercontent.com/dnzlike/CI-CD/master/sources/CI/NewToken.png)  

2. 进入所需要使用CI/CD环境的仓库下，添加webhook<br></br>
![avatar](https://raw.githubusercontent.com/dnzlike/CI-CD/master/sources/CI/%E5%88%9B%E5%BB%BAwebhook.png) 
![avatar](https://raw.githubusercontent.com/dnzlike/CI-CD/master/sources/CI/addwebhook.png) 

3. webhook显示连接成功<br></br>
![avatar](https://raw.githubusercontent.com/dnzlike/CI-CD/master/sources/CI/webhook%E8%BF%9E%E6%8E%A5%E6%88%90%E5%8A%9F.png)

### Jenkins系统管理
1. 在腾讯云服务器上(Ubuntu系统)下载Jenkins后用命令行打开，安装推荐插件，创建账号后进入Jenkins使用界面<br></br>
![avatar](https://raw.githubusercontent.com/dnzlike/CI-CD/master/sources/CI/%E8%BF%9B%E5%85%A5jenkins.png)

2. 安装 Github Plugin, Maven Integration Plungin 等插件<br></br>
3. 进入系统设置<br></br>
![avatar](https://raw.githubusercontent.com/dnzlike/CI-CD/master/sources/CI/%E7%B3%BB%E7%BB%9F%E8%AE%BE%E7%BD%AE.png)

* Github server配置<br></br>
![avatar](https://raw.githubusercontent.com/dnzlike/CI-CD/master/sources/CI/GithubServer.png)

* 点击Add,这里的secret就填写github提供的个人token<br></br>
![avatar](https://raw.githubusercontent.com/dnzlike/CI-CD/master/sources/CI/NewCredential.png)
![avatar](https://raw.githubusercontent.com/dnzlike/CI-CD/master/sources/CI/CreateCredentialWithToken.png)

* 完成系统设置  <br></br>

4. 进入全局工具配置<br></br>
![avatar](https://raw.githubusercontent.com/dnzlike/CI-CD/master/sources/CI/%E5%85%A8%E5%B1%80%E5%B7%A5%E5%85%B7%E9%85%8D%E7%BD%AE.png)

* Maven Configuration（以下几步均使用上一次CICD作业截图，实际文件路径为linux环境下的配置文件路径）<br></br>
![avatar](https://raw.githubusercontent.com/dnzlike/CI-CD/master/sources/CI/MavenConfigration.png)

* JDK安装<br></br>
![avatar](https://raw.githubusercontent.com/dnzlike/CI-CD/master/sources/CI/JDK.png)

* Git配置<br></br>
![avatar](https://raw.githubusercontent.com/dnzlike/CI-CD/master/sources/CI/Git.png)

* Maven安装<br></br>
![avatar](https://raw.githubusercontent.com/dnzlike/CI-CD/master/sources/CI/Maven.png)

* 完成全局工具配置<br></br>

### 构建项目
1. 在Jenkins中构建一个maven项目<br></br>
![avatar](https://raw.githubusercontent.com/dnzlike/CI-CD/master/sources/CI/CreateMavenProject.png)

2. 进入项目配置<br></br>
![avatar](https://raw.githubusercontent.com/dnzlike/CI-CD/master/sources/CI/General.png)

3. 使用Git进行源码管理，需提供github仓库URL和Github账号密码<br></br>
![avatar](https://raw.githubusercontent.com/dnzlike/CI-CD/master/sources/CI/%E6%BA%90%E7%A0%81%E7%AE%A1%E7%90%86.png)

4. 构建触发器,选择此项是为了利用webhook实现push后自动构建<br></br>
![avatar](https://raw.githubusercontent.com/dnzlike/CI-CD/master/sources/CI/%E6%9E%84%E5%BB%BA%E8%A7%A6%E5%8F%91%E5%99%A8.png)

5. 构建环境，仍使用Token授权<br></br>
![avatar](https://raw.githubusercontent.com/dnzlike/CI-CD/master/sources/CI/%E6%9E%84%E5%BB%BA%E7%8E%AF%E5%A2%83.png)

6. Build<br></br>
![avatar](https://raw.githubusercontent.com/dnzlike/CI-CD/master/sources/CI/Build.png)

7. 构建后操作<br></br>
![avatar](https://raw.githubusercontent.com/dnzlike/CI-CD/master/sources/CI/%E6%9E%84%E5%BB%BA%E5%90%8E%E6%93%8D%E4%BD%9C.png)

* 配置完成<br></br>

## 搭建CD环境
### *maven* 项目需要将打包方式设置为 .war 以进行后续部署。
1. 项目结构一览<br></br>
![project structure](https://raw.githubusercontent.com/dnzlike/CI-CD/master/sources/Project/maven%E9%A1%B9%E7%9B%AE%E7%BB%93%E6%9E%84.png)

2. 修改pom.xml

* 设置 .war 打包方式

        <packaging>war</packaging>
  ![war](https://raw.githubusercontent.com/dnzlike/CI-CD/master/sources/Project/jar%E6%94%B9war.png)

* 添加 *tomcat* 依赖

        <dependencies>

                ...   <!-- other dependencies -->

                <dependency>
                        <groupId>org.springframework.boot</groupId>
                        <artifactId>spring-boot-starter-tomcat</artifactId>
                        <scope>provided</scope>
                </dependency>
        </dependencies>

3. 修改启动类<br></br>
![SpringBootApp](https://raw.githubusercontent.com/dnzlike/CI-CD/master/sources/Project/%E4%BF%AE%E6%94%B9%E5%90%AF%E5%8A%A8%E7%B1%BB.png)

### 在项目通过CI（jenkins构建和测试）后，需要自动部署到web应用服务器（tomcat）上。
1. 启动 *tomcat*

        cd /d D:\apache-tomcat-9.0.7\bin
        startup tomcat
   ![tomcat](https://raw.githubusercontent.com/dnzlike/CI-CD/master/sources/CD/%E9%83%A8%E7%BD%B2tomcat.png)

2. *jenkins* 构建项目后打包成 .war 包，在 build 后执行 Windows batch command

        copy C:\Users\DNZ\\.jenkins\workspace\cicdmaven\target\demo-0.0.1-SNAPSHOT.war D:\apache-tomcat-9.0.7\webapps

    将 *jenkins* 打包的 war 包部署到本机 *tomcat* 的 /webapps 目录下
    <br></br>
![post](https://raw.githubusercontent.com/dnzlike/CI-CD/master/sources/CD/PostSteps.png)


3. *tomcat* 检测到 .war 包，进行部署
<br></br>
![tomcat success](https://raw.githubusercontent.com/dnzlike/CI-CD/master/sources/CD/tomcat%E9%83%A8%E7%BD%B2%E6%88%90%E5%8A%9F.png)

4. 打开浏览器，访问部署在 *8080* 端口的 *demo-0.0.1-SNAPSHOT* 项目，成功。<br></br>
![request](https://raw.githubusercontent.com/dnzlike/CI-CD/master/sources/CD/访问web.png)

## 测试效果
1. 修改项目并push<br></br>
![avatar](https://raw.githubusercontent.com/dnzlike/CI-CD/master/sources/pushtest/%E4%BF%AE%E6%94%B9%E9%A1%B9%E7%9B%AE%E5%B9%B6%E6%8F%90%E4%BA%A4.png)

2. Jenkins开始自动构建<br></br>
![avatar](https://raw.githubusercontent.com/dnzlike/CI-CD/master/sources/pushtest/Jenkins%E8%87%AA%E5%8A%A8%E6%9E%84%E5%BB%BA.png)

3. 构建成功并通过测试<br></br>
![avatar](https://raw.githubusercontent.com/dnzlike/CI-CD/master/sources/pushtest/%E6%9E%84%E5%BB%BA%E6%88%90%E5%8A%9F%E6%B5%8B%E8%AF%95%E9%80%9A%E8%BF%87.png)
![avatar](https://raw.githubusercontent.com/dnzlike/CI-CD/master/sources/pushtest/%E6%B5%8B%E8%AF%95%E9%80%9A%E8%BF%87.png)

4. Github hook日志<br></br>
![avatar](https://raw.githubusercontent.com/dnzlike/CI-CD/master/sources/pushtest/GithubHookLog.png)

5. 成功部署，查看修改后页面<br></br>
![avatar](https://raw.githubusercontent.com/dnzlike/CI-CD/master/sources/pushtest/%E6%9F%A5%E7%9C%8B%E4%BF%AE%E6%94%B9%E5%90%8E%E7%9A%84web%E9%A1%B5%E9%9D%A2.png)
