# Day5-用户接口实现和后台页面骨架

## 用户权限

### 动态获取用户权限

#### 创建用户权限表 `t_user_role`

在 mysql 数据库管理工具中在数据库执行如下语句创建新表：

```sql
CREATE TABLE `t_user_role` (
  `id` bigint(20) UNSIGNED NOT NULL COMMENT 'id',
  `username` varchar(60) NOT NULL COMMENT '用户名',
  `role` varchar(60) NOT NULL COMMENT '角色',
  `create_time` datetime NOT NULL DEFAULT current_timestamp() COMMENT '创建时间'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci COMMENT='用户角色表' ROW_FORMAT=DYNAMIC;


ALTER TABLE `t_user_role`
  ADD PRIMARY KEY (`id`) USING BTREE,
  ADD KEY `idx_username` (`username`) USING BTREE;


ALTER TABLE `t_user_role`
  MODIFY `id` bigint(20) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT 'id';
COMMIT;
```

创建完成后得到如下结构的表

![t\_user\_role 表](<.gitbook/assets/image 20241115103856673.png>)

#### 在项目中创建数据表结构和映射

首先在 `common.domain.dos` 包下创建 `UserRoleDO` 类

```java
package cn.dogalist.weblog.common.domain.dos;

import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableName;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.util.Date;

/**
 * description: 用户角色关联表
 */

@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
@TableName("t_user_role")
public class UserRoleDO {
    @TableId(type = IdType.AUTO)
    private Long id;
    private String username;
    private String role;
    private Date createTime;
}
```

在 `common.domain.mapper` 包下创建 `UserRoleMapper` 类，构建一个通过用户名查询数据条目的方法：

```java
package cn.dogalist.weblog.common.domain.mapper;
import cn.dogalist.weblog.common.domain.dos.UserRoleDO;
import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import java.util.List;
public interface UserRoleMapper extends BaseMapper<UserRoleDO> {
    default List<UserRoleDO> findByUsername(String username)
    {
        LambdaQueryWrapper<UserRoleDO> queryWrapper = new LambdaQueryWrapper<>();
        queryWrapper.eq(UserRoleDO::getUsername, username);
        return selectList(queryWrapper);
    }
}
```

#### 修改 Security 配置

对之前访问权限不足处理器 `RestAccessDeniedHandler` 类的留白部分进行补充：

```java
@Component
public class RestAccessDeniedHandler implements AccessDeniedHandler {
    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response,
            AccessDeniedException accessDeniedException) throws IOException, ServletException {
        log.warn("登录成功访问受保护的资源，但是权限不足: ", accessDeniedException);
        // todo 预留，后面引入多角色时会用到
        ResultUtil.fail(response,Response.fail(ResponseCodeEnum.FORBIDDEN));
    }
}
```

同时需要在 `ResponseCodeEnum` 中添加新的异常状态码：

```java
    // ----------- 权限异常状态码 -----------
    FORBIDDEN("20006", "演示账号仅支持查询操作！"),
```

在 `admin.config` 下的 `WebSecurityConfig` 配置类中开启 Security 的一些注解，同时启用访问权限不足的处理器 `RestAccessDeniedHandler`。

```java
import cn.dogalist.weblog.jwt.config.JwtAuthenticationSecurityConfig;
import cn.dogalist.weblog.jwt.filter.TokenAuthenticationFilter;
import cn.dogalist.weblog.jwt.handler.RestAccessDeniedHandler;
import cn.dogalist.weblog.jwt.handler.RestAuthenticationEntryPoint;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
@@ -18,11 +20,14 @@
 */
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true)
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Autowired
    private JwtAuthenticationSecurityConfig jwtAuthenticationSecurityConfig;
    @Autowired
    private RestAuthenticationEntryPoint authEntryPoint;
    @Autowired
    private RestAccessDeniedHandler accessDeniedHandler;
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // 禁用CSRF
@@ -36,6 +41,8 @@ protected void configure(HttpSecurity http) throws Exception {
                .and()
                .httpBasic().authenticationEntryPoint(authEntryPoint)
                .and()
                .exceptionHandling().accessDeniedHandler(accessDeniedHandler) //添加自定义的访问权限不足处理器
                .and()
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS) // 前后端分离，无需创建会话
                .and()
                .addFilterBefore(tokenAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class)
```

在 `@EnableGlobalMethodSecurity` 中, 设置 `prePostEnabled = true` 可以启用 `@PreAuthorize` 和 `@PostAuthorize` 注解。

`@PreAuthorize` 注解在方法执行前进行权限验证，而 `@PostAuthorize` 注解在方法执行后进行权限验证。

设置 `securedEnabled = true` 可以启用 `@Secured` 注解。`@Secured` 注解用于定义业务方法的安全配置，只有那些具有指定角色的用户才可以调用该方法。例如：

```java
@Secured({"ROLE_USER"})

void updateUser(User user);

@Secured({"ROLE_ADMIN", "ROLE_USER1"})

void deleteUser();
```

> **注意:** **@Secured** 注解不支持 Spring EL 表达式，指定的角色必须以 **ROLE\_** 开头。

对于权限不足的情况，我们还要手动抛出错误。在 `common.exception` 包下的全局异常处理类 `GlobalExceptionHandler` 中定义方法：

```java
@ExceptionHandler(AccessDeniedException.class)
    public void throwAccessDeniedException(AccessDeniedException e) throws AccessDeniedException {
        // 捕获到鉴权失败异常，主动抛出，交给 RestAccessDeniedHandler 去处理
        log.info("============= 捕获到 AccessDeniedException");
        throw e;
    }

```

> **注意:** 捕获该类需要在 `common` 模块下添加 Security 依赖，在 `weblog-module-common` 的 `pom.xml` 配置文件中添加依赖：
>
> ```xml
> <dependencies>        
>     <dependency>
>         <groupId>org.springframework.boot</groupId>
>         <artifactId>spring-boot-starter-security</artifactId>
>     </dependency>
>         <!-- 其他内容 -->
>   <dependencies>
> ```

#### 修改用户详细服务接口

重新写一下 `jwt.service` 中的 `UserDetailServiceImpl`，我们已经建立了连接数据库获取用户权限的逻辑，接下来将他运用到接口中：

```java
/**
 * 用户信息获取类
 */
@Service
@Slf4j
public class UserDetailServiceImpl implements UserDetailsService {
    @Autowired
    private UserMapper userMapper;
    @Autowired
    private UserRoleMapper userRoleMapper;
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {

        // 从数据库中查询
        UserDO userDO = userMapper.findByUsername(username);

        // 判断用户是否存在
        if (Objects.isNull(userDO)) {
            throw new UsernameNotFoundException("该用户不存在");
        }

        // 查询权限
        List<UserRoleDO> userRoleDO = userRoleMapper.findByUsername(username);
        // 转数组
        List<String> roles = userRoleDO.stream().map(UserRoleDO::getRole).collect(Collectors.toList());
        String[] roleArr = roles.toArray(new String[roles.size()]);
//        log.info(Arrays.toString(roleArr));
        return User.withUsername(userDO.getUsername())
                .password(userDO.getPassword())
                .authorities(roleArr)
                .build();
    }
```

#### 测试权限

在 `TestController` 中新建测试接口：

```java
    @PostMapping("/admin/update")
    @ApiOperationLog(description = "测试更新接口")
    @ApiOperation(value = "测试更新接口")
    @PreAuthorize("hasRole('ROLE_ADMIN')")
    public Response testUpdate() {
        log.info("更新成功...");
        return Response.success();
    }
```

同时在 `t_user` 表中新建用户 `test`, 密码使用 `jwt` 模块下 `PasswordEncoderConfig` 中的方法加密一下之后填入。然后在 `t_user_role` 表中设定用户的权限，给管理员账号设定 `ROLE_ADMIN` 的角色，给新建的 `test` 用户设定 `ROLE_VISITOR` 角色。最后数据项如下：

![用户权限数据条目](<.gitbook/assets/image 20241115184444861.png>)

可以在 api 文档中进行调试，登录对应账户获取 token，添加到请求头中看看是否正常地验证了用户权限

## 获取用户信息和修改密码

### 构建请求和响应 VO

#### 获取用户信息响应

获取一个用户的信息，只需要通过用户名来请求，所以重点需要构建响应的 VO，在 `admin.model.vo.user` 下新建 `FindUserInfoRspVO` 类，对于请求用户信息的响应，我们目前定义是返回其用户名和角色列表，以便在前端显示：

```java
package cn.dogalist.weblog.admin.model.vo.user;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class FindUserInfoRspVO {
    private String username;
    private String[] roles;
}
```

#### 修改密码请求 VO

请求一次修改密码，需要用户名、原密码和旧密码三项内容，在 `admin.model.vo` 下新建 `UpdateAdminUserPasswordReqVO` 类：

```java
package cn.dogalist.weblog.admin.model.vo;
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import javax.validation.constraints.NotBlank;
@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
@ApiModel(value = "修改用户密码 VO", description = "修改密码请求对象")
public class UpdateAdminUserPasswordReqVO {
    @NotBlank(message = "用户名不能为空")
    @ApiModelProperty(value = "用户名", required = true)
    private String username;
    @NotBlank(message = "旧密码不能为空")
    @ApiModelProperty(value = "旧密码", required = true)
    private String oldPassword;
    @NotBlank(message = "密码不能为空")
    @ApiModelProperty(value = "密码", required = true)
    private String password;
}
```

同时需要引入参数验证所需的依赖包，在 `admin` 模块下的 `pom.xml` 中添加以下依赖：

```xml
<dependency>
    <groupId>jakarta.validation</groupId>
    <artifactId>jakarta.validation-api</artifactId>
</dependency>
```

### 构建服务接口

#### 创建接口

在 `admin.service` 下新建接口 `AdminUserService`：

```java
package cn.dogalist.weblog.admin.service;
import cn.dogalist.weblog.admin.model.vo.UpdateAdminUserPasswordReqVO;
import cn.dogalist.weblog.common.utils.Response;
public interface AdminUserService {
    /**
     * 修改密码
     * @param updateAdminUserPasswordReqVO
     * @return
     */
    Response updatePassword(UpdateAdminUserPasswordReqVO updateAdminUserPasswordReqVO);
    Response findUserInfo();
}
```

#### 实现接口

接下来对两个接口进行实现。在 `admin.service.impl` 中创建类 `AdminUserServiceImpl`:

```java
package cn.dogalist.weblog.admin.service.impl;
import cn.dogalist.weblog.admin.model.vo.UpdateAdminUserPasswordReqVO;
import cn.dogalist.weblog.admin.model.vo.user.FindUserInfoRspVO;
import cn.dogalist.weblog.admin.service.AdminUserService;
import cn.dogalist.weblog.common.domain.dos.UserDO;
import cn.dogalist.weblog.common.domain.dos.UserRoleDO;
import cn.dogalist.weblog.common.domain.mapper.UserMapper;
import cn.dogalist.weblog.common.domain.mapper.UserRoleMapper;
import cn.dogalist.weblog.common.enums.ResponseCodeEnum;
import cn.dogalist.weblog.common.utils.Response;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;
import java.util.List;
import java.util.stream.Collectors;
@Service
public class AdminUserServiceImpl implements AdminUserService {
    @Autowired
    private UserMapper userMapper;
    @Autowired
    private UserRoleMapper userRoleMapper;
    @Autowired
    private PasswordEncoder passwordEncoder;
    @Override
    public Response updatePassword(UpdateAdminUserPasswordReqVO updateAdminUserPasswordReqVO) {
        // 获取用户、旧密码、密码
        String userName = updateAdminUserPasswordReqVO.getUsername();
        String oldPassword = updateAdminUserPasswordReqVO.getOldPassword();
        String password = updateAdminUserPasswordReqVO.getPassword();
        UserDO userDO = userMapper.findByUsername(userName);
        // 用户是否存在
        if (userDO == null) {
            return Response.fail(ResponseCodeEnum.USERNAME_NOT_FOUND);
        }
        // 校验旧密码是否正确
        if (!passwordEncoder.matches(oldPassword, userDO.getPassword())) {
            return Response.fail(ResponseCodeEnum.PASSWORD_ERROR);
        }
        // 新旧密码一致
        if (oldPassword.equals(password)) {
            return Response.fail(ResponseCodeEnum.PASSWORD_NOT_CHANGE);
        }
        // 加密
        String encryptPassword = passwordEncoder.encode(password);
        // 更新密码
        int count = userMapper.updatePasswordByUsername(userName, encryptPassword);
        return count == 1 ? Response.success() : Response.fail(ResponseCodeEnum.SYSTEM_ERROR);
    }
    @Override
    public Response findUserInfo() {
        // 获取存储在ThreadLocal中的用户名
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        String username = authentication.getName();
        // 获取用户权限角色
        List<UserRoleDO> userRoleDO = userRoleMapper.findByUsername(username);
        // 封装成字符串数组
        String[] roleArr = userRoleDO.stream().map(UserRoleDO::getRole).collect(Collectors.toList()).toArray(new String[userRoleDO.size()]);
        return Response.success(FindUserInfoRspVO.builder().username(username).roles(roleArr).build());
    }
}
```

之前在验证中，我们将用户名存储到了 ThreadLocal 中，在实现获取用户信息的接口时直接调用即可。对于修改密码中的几种错误情况，我们也需要定义相应的异常状态码。在异常状态码枚举类中添加定义：

```java
USERNAME_NOT_FOUND("20003", "该用户不存在"),
PASSWORD_ERROR("20005", "用户密码错误"),
PASSWORD_NOT_CHANGE("20006","新旧密码不能相等" );
```

### 定义对应的 api

在 `admin.controller` 下创建 `AdminUserController` 类：

```java
package cn.dogalist.weblog.admin.controller;
import cn.dogalist.weblog.admin.model.vo.UpdateAdminUserPasswordReqVO;
import cn.dogalist.weblog.admin.service.AdminUserService;
import cn.dogalist.weblog.common.aspect.ApiOperationLog;
import cn.dogalist.weblog.common.utils.Response;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/admin")
@Api(tags = "Admin 用户模块")
public class AdminUserController {
    @Autowired
    private AdminUserService userService;
    @PostMapping("/password/update")
    @ApiOperation(value = "修改密码")
    @ApiOperationLog(description = "修改用户密码")
    public Response updatePassword(@RequestBody @Validated UpdateAdminUserPasswordReqVO updateAdminUserPasswordReqVO)
    {
        return userService.updatePassword(updateAdminUserPasswordReqVO);
    }
    @PostMapping("/user/info")
    @ApiOperation(value = "获取用户信息")
    @ApiOperationLog(description = "获取用户信息")
    public Response findUserInfo()
    {
        return userService.findUserInfo();
    }
}
```

## 后台骨架的搭建

搭建的时候写的太上头忘记写文档了，代码都是全写完之后提交的，没法一点点来说，直接看提交记录吧各位🥺

[后台页面骨架搭建 · monthwolf/weblog-vue3@f75a413](https://github.com/monthwolf/weblog-vue3/commit/f75a41312be205b63bdf437b8873d07a1d5260de)

[暗色模式 · monthwolf/weblog-vue3@0e424f4](https://github.com/monthwolf/weblog-vue3/commit/0e424f4f775610d2d2d564531f5c43315c2c2f09)

[获取用户信息和修改密码 · monthwolf/weblog-vue3@64d7998](https://github.com/monthwolf/weblog-vue3/commit/64d7998872eeb2f78020fc970bd5f37ab3d1c96a)

[对接后端api · monthwolf/weblog-vue3@f61948d](https://github.com/monthwolf/weblog-vue3/commit/f61948da1b78bef2a8ae5a4a9fc1142f0e6d7f6f)
