### 概述

在实际开发中，我们的用户信息都是存在数据库里的，本章节基于 **RBAC 模型** 将用户的认证信息与数据库对接，实现真正的用户认证与授权

**操作流程**

继续 **基于 JDBC 存储令牌** 章节的代码开发

- 初始化 RBAC 相关表

- 在数据库中配置“用户”、“角色”、“权限”相关信息

- 数据库操作使用 **tk.mybatis** 框架，故需要增加相关依赖

- 配置 Web 安全

    - 配置使用自定义认证与授权

- 通过 GET 请求访问认证服务器获取授权码

    - 端点：**/oauth/authorize**

- 通过 POST 请求利用授权码访问认证服务器获取令牌

    - 端点：**/oauth/token**

**默认的端点 URL**

- **/oauth/authorize**：授权端点

- **/oauth/token**：令牌端点

- **/oauth/confirm_access**：用户确认授权提交端点

- **/oauth/error**：授权服务错误信息端点

- **/oauth/check_token**：用于资源服务访问的令牌解析端点

- **/oauth/token_key**：提供公有密匙的端点，如果你使用 JWT 令牌的话

### 初始化 RBAC 相关表

```
CREATE TABLE `tb_permission` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `parent_id` bigint(20) DEFAULT NULL COMMENT '父权限',
  `name` varchar(64) NOT NULL COMMENT '权限名称',
  `enname` varchar(64) NOT NULL COMMENT '权限英文名称',
  `url` varchar(255) NOT NULL COMMENT '授权路径',
  `description` varchar(200) DEFAULT NULL COMMENT '备注',
  `created` datetime NOT NULL,
  `updated` datetime NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=44 DEFAULT CHARSET=utf8 COMMENT='权限表';
insert  into `tb_permission`(`id`,`parent_id`,`name`,`enname`,`url`,`description`,`created`,`updated`) values 
(37,0,'系统管理','System','/',NULL,'2019-04-04 23:22:54','2019-04-04 23:22:56'),
(38,37,'用户管理','SystemUser','/users/',NULL,'2019-04-04 23:25:31','2019-04-04 23:25:33'),
(39,38,'查看用户','SystemUserView','',NULL,'2019-04-04 15:30:30','2019-04-04 15:30:43'),
(40,38,'新增用户','SystemUserInsert','',NULL,'2019-04-04 15:30:31','2019-04-04 15:30:44'),
(41,38,'编辑用户','SystemUserUpdate','',NULL,'2019-04-04 15:30:32','2019-04-04 15:30:45'),
(42,38,'删除用户','SystemUserDelete','',NULL,'2019-04-04 15:30:48','2019-04-04 15:30:45');

CREATE TABLE `tb_role` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `parent_id` bigint(20) DEFAULT NULL COMMENT '父角色',
  `name` varchar(64) NOT NULL COMMENT '角色名称',
  `enname` varchar(64) NOT NULL COMMENT '角色英文名称',
  `description` varchar(200) DEFAULT NULL COMMENT '备注',
  `created` datetime NOT NULL,
  `updated` datetime NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=38 DEFAULT CHARSET=utf8 COMMENT='角色表';
insert  into `tb_role`(`id`,`parent_id`,`name`,`enname`,`description`,`created`,`updated`) values 
(37,0,'超级管理员','admin',NULL,'2019-04-04 23:22:03','2019-04-04 23:22:05');

CREATE TABLE `tb_role_permission` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `role_id` bigint(20) NOT NULL COMMENT '角色 ID',
  `permission_id` bigint(20) NOT NULL COMMENT '权限 ID',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=43 DEFAULT CHARSET=utf8 COMMENT='角色权限表';
insert  into `tb_role_permission`(`id`,`role_id`,`permission_id`) values 
(37,37,37),
(38,37,38),
(39,37,39),
(40,37,40),
(41,37,41),
(42,37,42);

CREATE TABLE `tb_user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `username` varchar(50) NOT NULL COMMENT '用户名',
  `password` varchar(64) NOT NULL COMMENT '密码，加密存储',
  `phone` varchar(20) DEFAULT NULL COMMENT '注册手机号',
  `email` varchar(50) DEFAULT NULL COMMENT '注册邮箱',
  `created` datetime NOT NULL,
  `updated` datetime NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `username` (`username`) USING BTREE,
  UNIQUE KEY `phone` (`phone`) USING BTREE,
  UNIQUE KEY `email` (`email`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=38 DEFAULT CHARSET=utf8 COMMENT='用户表';
insert  into `tb_user`(`id`,`username`,`password`,`phone`,`email`,`created`,`updated`) values 
(37,'admin','$2a$10$9ZhDOBp.sRKat4l14ygu/.LscxrMUcDAfeVOEPiYwbcRkoB09gCmi','15888888888','lee.lusifer@gmail.com','2019-04-04 23:21:27','2019-04-04 23:21:29');

CREATE TABLE `tb_user_role` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `user_id` bigint(20) NOT NULL COMMENT '用户 ID',
  `role_id` bigint(20) NOT NULL COMMENT '角色 ID',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=38 DEFAULT CHARSET=utf8 COMMENT='用户角色表';
insert  into `tb_user_role`(`id`,`user_id`,`role_id`) values 
(37,37,37);
```

由于使用了 **BCryptPasswordEncoder** 的加密方式，故用户密码需要加密，代码如下：

```
System.out.println(new BCryptPasswordEncoder().encode("123456"));
```

### POM

数据库操作采用 **tk.mybatis:mapper-spring-boot-starter:2.1.5** 框架，需增加相关依赖，完整 POM 如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.funtl</groupId>
        <artifactId>hello-spring-security-oauth2</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>

    <artifactId>hello-spring-security-oauth2-server</artifactId>
    <url>http://www.funtl.com</url>

    <licenses>
        <license>
            <name>Apache 2.0</name>
            <url>https://www.apache.org/licenses/LICENSE-2.0.txt</url>
        </license>
    </licenses>

    <developers>
        <developer>
            <id>liwemin</id>
            <name>Lusifer Lee</name>
            <email>lee.lusifer@gmail.com</email>
        </developer>
    </developers>

    <dependencies>
        <!-- Spring Boot -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>

        <!-- Spring Security -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-oauth2</artifactId>
        </dependency>

        <!-- CP -->
        <dependency>
            <groupId>com.zaxxer</groupId>
            <artifactId>HikariCP</artifactId>
            <version>${hikaricp.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
            <exclusions>
                <!-- 排除 tomcat-jdbc 以使用 HikariCP -->
                <exclusion>
                    <groupId>org.apache.tomcat</groupId>
                    <artifactId>tomcat-jdbc</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${mysql.version}</version>
        </dependency>
        <dependency>
            <groupId>tk.mybatis</groupId>
            <artifactId>mapper-spring-boot-starter</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>com.funtl.spring.security.oauth2.server.OAuth2ServerApplication</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
    
</project>
```

### application.yml

```
spring:
  application:
    name: oauth2-server
  datasource:
    type: com.zaxxer.hikari.HikariDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    jdbc-url: jdbc:mysql://192.168.141.130:3306/oauth2?useUnicode=true&characterEncoding=utf-8&useSSL=false
    username: root
    password: 123456
    hikari:
      minimum-idle: 5
      idle-timeout: 600000
      maximum-pool-size: 10
      auto-commit: true
      pool-name: MyHikariCP
      max-lifetime: 1800000
      connection-timeout: 30000
      connection-test-query: SELECT 1
      
server:
  port: 8080
  
mybatis:
  type-aliases-package: com.funtl.spring.security.oauth2.server.domain
  mapper-locations: classpath:mapper/*.xml
```

### Application

增加了 Mapper 的包扫描配置

```
package com.funtl.spring.security.oauth2.server;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import tk.mybatis.spring.annotation.MapperScan;

@SpringBootApplication
@MapperScan(basePackages = "com.funtl.spring.security.oauth2.server.mapper")
public class OAuth2ServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(OAuth2ServerApplication.class, args);
    }
}
```

### 关键步骤

由于本次增加了 MyBatis 相关操作，代码增加较多，可以参考我 [GitHub](https://github.com/topsale/spring-boot-samples/tree/master/spring-security-oauth2) 上的源码，下面仅列出关键步骤及代码

### 获取用户信息

目的是为了实现自定义认证授权时可以通过数据库查询用户信息，Spring Security oAuth2 要求使用 **username** 的方式查询，提供相关用户信息后，认证工作由框架自行完成

```
package com.funtl.spring.security.oauth2.server.service.impl;

import com.funtl.spring.security.oauth2.server.domain.TbUser;
import com.funtl.spring.security.oauth2.server.mapper.TbUserMapper;
import com.funtl.spring.security.oauth2.server.service.TbUserService;
import org.springframework.stereotype.Service;
import tk.mybatis.mapper.entity.Example;
import javax.annotation.Resource;

@Service
public class TbUserServiceImpl implements TbUserService {

    @Resource
    private TbUserMapper tbUserMapper;
    
    @Override
    public TbUser getByUsername(String username) {
        Example example = new Example(TbUser.class);
        example.createCriteria().andEqualTo("username", username);
        return tbUserMapper.selectOneByExample(example);
    }
}
```

### 获取用户权限信息

认证成功后需要给用户授权，具体的权限已经存储在数据库里了

```
package com.funtl.spring.security.oauth2.server.mapper;

import com.funtl.spring.security.oauth2.server.domain.TbPermission;
import org.apache.ibatis.annotations.Param;
import tk.mybatis.mapper.MyMapper;

import java.util.List;

public interface TbPermissionMapper extends MyMapper<TbPermission> {

    List<TbPermission> selectByUserId(@Param("id") Long id);
    
}
```

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.funtl.spring.security.oauth2.server.mapper.TbPermissionMapper">
  <resultMap id="BaseResultMap" type="com.funtl.spring.security.oauth2.server.domain.TbPermission">
    <!--@mbg.generated generated on Tue Jul 16 00:41:48 CST 2019.-->
    <id column="id" jdbcType="BIGINT" property="id" />
    <result column="parent_id" jdbcType="BIGINT" property="parentId" />
    <result column="name" jdbcType="VARCHAR" property="name" />
    <result column="enname" jdbcType="VARCHAR" property="enname" />
    <result column="url" jdbcType="VARCHAR" property="url" />
    <result column="description" jdbcType="VARCHAR" property="description" />
    <result column="created" jdbcType="TIMESTAMP" property="created" />
    <result column="updated" jdbcType="TIMESTAMP" property="updated" />
  </resultMap>
  <sql id="Base_Column_List">
    <!--@mbg.generated generated on Tue Jul 16 00:41:48 CST 2019.-->
    id, parent_id, `name`, enname, url, description, created, updated
  </sql>
  <select id="selectByUserId" resultMap="BaseResultMap">
    SELECT
      p.*
    FROM
      tb_user AS u
      LEFT JOIN tb_user_role AS ur
        ON u.id = ur.user_id
      LEFT JOIN tb_role AS r
        ON r.id = ur.role_id
      LEFT JOIN tb_role_permission AS rp
        ON r.id = rp.role_id
      LEFT JOIN tb_permission AS p
        ON p.id = rp.permission_id
    WHERE u.id = #{id}
  </select>
</mapper>
```

### 自定义认证授权实现类

创建一个类，实现 **UserDetailsService** 接口，代码如下：

```
package com.funtl.spring.security.oauth2.server.configure;

import com.funtl.spring.security.oauth2.server.domain.TbPermission;
import com.funtl.spring.security.oauth2.server.domain.TbUser;
import com.funtl.spring.security.oauth2.server.service.TbPermissionService;
import com.funtl.spring.security.oauth2.server.service.TbUserService;
import org.assertj.core.util.Lists;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class UserDetailsServiceImpl implements UserDetailsService {

    @Autowired
    private TbUserService tbUserService;
    
    @Autowired
    private TbPermissionService tbPermissionService;
    
    @Override
    public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
        TbUser tbUser = tbUserService.getByUsername(s);
        List<GrantedAuthority> grantedAuthorities = Lists.newArrayList();
        
        if (tbUser != null) {
            // 声明用户授权
            List<TbPermission> tbPermissions = tbPermissionService.selectByUserId(tbUser.getId());
            tbPermissions.forEach(tbPermission -> {
                GrantedAuthority grantedAuthority = new SimpleGrantedAuthority(tbPermission.getEnname());
                grantedAuthorities.add(grantedAuthority);
            });
            
            // 由框架完成认证工作
            return new User(tbUser.getUsername(), tbUser.getPassword(), grantedAuthorities);
        }
        
        return null;
    }
}
```

### 服务器安全配置

创建一个类继承 **WebSecurityConfigurerAdapter** 并添加相关注解：

- **@Configuration**

- **@EnableWebSecurity**

- **@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true, jsr250Enabled = true)**：全局方法拦截

```
package com.funtl.spring.security.oauth2.server.configure;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.WebSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true, jsr250Enabled = true)
public class WebSecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Bean
    public BCryptPasswordEncoder passwordEncoder() {
        // 配置默认的加密方式
        return new BCryptPasswordEncoder();
    }

    @Bean
    @Override
    protected UserDetailsService userDetailsService() {
        return new UserDetailsServiceImpl();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        // 使用自定义认证与授权
        auth.userDetailsService(userDetailsService());
    }

    @Override
    public void configure(WebSecurity web) throws Exception {
        // 将 check_token 暴露出去，否则资源服务器访问时报 403 错误
        web.ignoring().antMatchers("/oauth/check_token");
    }
}
```

### 访问获取授权码

- 通过浏览器访问

```
http://localhost:8080/oauth/authorize?client_id=client&response_type=code
```

- 第一次访问会跳转到登录页面

![](../img/11-00000008.png)

- 验证成功后会询问用户是否授权客户端

![](../img/11-00000009.png)

- 选择授权后会跳转到百度，浏览器地址上还会包含一个授权码（**code=1JuO6V**），浏览器地址栏会显示如下地址：

```
https://www.baidu.com/?code=1JuO6V
```

### 向服务器申请令牌

- 通过 CURL 或是 Postman 请求

```
curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d 'grant_type=authorization_code&code=1JuO6V'
"http://client:secret@localhost:8080/oauth/token"
```

![](../img/11-00000010.png)

- 得到响应结果如下

```
{
    "access_token": "016d8d4a-dd6e-4493-b590-5f072923c413",
    "token_type": "bearer",
    "expires_in": 43199,
    "scope": "app"
}
```

操作成功后数据库 **oauth_access_token** 表中会增加一笔记录，效果图如下：

![](../img/11-00000011.png)
