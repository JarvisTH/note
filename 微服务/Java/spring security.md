## 一、核心组件

- `SecurityContextHolder`，提供`SecurityContext`的访问权限。
- `SecurityContext`，保存`Authentication`和可能的特定于请求的安全信息。
- `Authentication`，以特定于Spring Security的方式代表校长。
- `GrantedAuthority`，以反映授予主体的应用程序范围的权限。
- `UserDetails`，提供从应用程序的DAO或其他安全数据源构建Authentication对象所需的信息。
- `UserDetailsService`，在基于`String`的用户名（或证书ID等）中传递时创建`UserDetails`。

### 1.SecurityContextHolder

最基本的对象是`SecurityContextHolder`，存储应用程序当前安全上下文的详细信息的地方，其中包括当前使用该应用程序的主体的详细信息。默认情况下，`SecurityContextHolder`使用`ThreadLocal`来存储这些详细信息，这意味着安全上下文始终可用于同一执行线程中的方法。

某些应用程序并不完全适合使用`ThreadLocal`，因为它们使用线程的特定方式，`SecurityContextHolder`可以在启动时配置策略，以指定您希望如何存储上下文。您可以通过两种方式从默认`SecurityContextHolder.MODE_THREADLOCAL`更改模式。第一个是设置系统属性，第二个是在`SecurityContextHolder`上调用静态方法。

### 2.Authentication

使用`Authentication`对象来表示此信息，通常不需要自己创建`Authentication`对象。

```java
Object principal = SecurityContextHolder.getContext().getAuthentication().getPrincipal();

if (principal instanceof UserDetails) {
	String username = ((UserDetails)principal).getUsername();
} else {
	String username = principal.toString();
}
```

调用`getContext()`返回的对象是`SecurityContext`接口的实例,Spring Security中的大多数认证机制都返回`UserDetails`的实例作为主体。

### 3.UserDetailsService

从`Authentication`对象获取主体。只是`Object`。大多数情况下，这可以转换为`UserDetails`对象。

什么时候提供`UserDetails`对象?   UserDetailsService此接口上唯一的方法接受基于`String`的用户名参数并返回`UserDetails`。

`UserDetailsService`纯粹是用户数据的DAO，除了将数据提供给框架内的其他组件之外，不执行任何其他功能。特别是，它*不会*对用户进行身份验证，这是由`AuthenticationManager`完成的。在许多情况下，如果您需要自定义身份验证过程，直接[实现`AuthenticationProvider`](https://www.springcloud.cc/spring-security.html#core-services-authentication-manager)会更有意义。

### 4.GrantedAuthority

`Authentication`提供的另一个重要方法是`getAuthorities()`。此方法提供`GrantedAuthority`个对象的数组。`GrantedAuthority`是授予权力。这些权力通常是“角色”，例如`ROLE_ADMINISTRATOR`或`ROLE_HR_SUPERVISOR`。`GrantedAuthority`对象通常由`UserDetailsService`加载。通常`GrantedAuthority`对象是应用程序范围的权限。它们不是特定于给定的对象。

## 二、认证

### 1.标准身份验证方案

1. 提示用户使用用户名和密码登录。
2. 系统（成功）验证用户名的密码是否正确。
3. 获取该用户的上下文信息（他们的角色列表等）。
4. 为用户建立安全上下文
5. 用户继续进行，可能执行一些可能受访问控制机制保护的操作，该访问控制机制针对当前安全上下文信息检查操作所需的许可。

前三项构成了身份验证过程，在Spring Security内过程如下：

1. 获取用户名和密码并将其合并到`UsernamePasswordAuthenticationToken`的实例中。
2. 令牌被传递给`AuthenticationManager`的实例以进行验证。
3. `AuthenticationManager`在成功验证后返回完全填充的`Authentication`实例。
4. 通过调用`SecurityContextHolder.getContext().setAuthentication(…)`建立安全上下文，传入返回的身份验证对象。

```java
UsernamePasswordAuthenticationToken authenticationToken =
                new UsernamePasswordAuthenticationToken(authUser.getUsername(), password);
Authentication authentication = authenticationManagerBuilder.getObject().authenticate(authenticationToken);
SecurityContextHolder.getContext().setAuthentication(authentication);
```

从此时起，用户被认为是经过身份验证的。

### 2.Web应用程序中的身份验证

典型的web应用程序的身份验证过程：

1. 您访问主页，然后单击链接。
2. 请求转到服务器，服务器确定您已请求受保护的资源。
3. 由于您目前尚未通过身份验证，因此服务器会发回一个响应，指示您必须进行身份验证。响应将是HTTP响应代码，或重定向到特定的web页面。
4. 根据身份验证机制，您的浏览器将重定向到特定的web页面，以便您可以填写表单，或者浏览器将以某种方式检索您的身份（通过BASIC身份验证对话框，cookie，X. 509证书等）。
5. 浏览器将向服务器发回响应。这将是包含您填写的表单内容的HTTP POST，或者包含您的身份验证详细信息的HTTP标头。
6. 接下来，服务器将决定所呈现的凭证是否有效。如果它们有效，则下一步将会发生。如果它们无效，通常会要求您的浏览器再次尝试（因此您将返回上面的第二步）。
7. 将重试您进行身份验证过程的原始请求。希望您已通过足够授权的权限进行身份验证以访问受保护资源。如果您有足够的访问权限，请求将成功。否则，您将收到HTTP错误代码403，这意味着“禁止”。

主要参与者（按照他们使用的顺序）是`ExceptionTranslationFilter`，`AuthenticationEntryPoint`和“认证机制”，它负责调用`AuthenticationManager`。

- #### ExceptionTranslationFilter

一个Spring Security过滤器，负责检测抛出的任何Spring Security异常。`AbstractSecurityInterceptor`通常会抛出此类异常，授权服务的主要提供者，产生Java异常。

- #### AuthenticationEntryPoint

负责上面列表中的第三步，每个主要身份验证系统都有自己的`AuthenticationEntryPoint`实现。

- #### 认证机制

基于表单的登录和基本身份验证。一旦从用户代理收集了身份验证详细信息，就会构建一个`Authentication`“请求”对象，然后将其呈现给`AuthenticationManager`。在认证机制收到完全填充的`Authentication`对象后，它将认为请求有效，将`Authentication`放入`SecurityContextHolder`，并使原始请求重试（7）。如果`Authe               nticationManager`拒绝了请求，则认证机制将要求用户代理重试（上面的步骤2）。

- #### 在请求之间存储SecurityContext

在典型的web应用程序中，用户登录一次，然后生成会话ID标识，服务器缓存持续时间会话的主体信息。在请求之间存储`SecurityContext`的责任落在`SecurityContextPersistenceFilter`上，默认情况下，该上下文将上下文存储为HTTP请求之间的`HttpSession`属性。它会为每个请求恢复上下文`SecurityContextHolder`，并且最重要的是，在请求完成时清除`SecurityContextHolder`。

出于安全考虑，不应直接与`HttpSession`进行交互 - 总是使用`SecurityContextHolder`。

## 三、授权

### 1.AbstractSecurityInterceptor

负责在Spring Security中做出访问控制决策是`AccessDecisionManager`。有一个`decide`方法，它接受一个代表请求访问的主体的`Authentication`对象。

Spring Security使用该术语来指代可以对其应用安全性（例如授权决策）的任何对象。最常见的示例是方法调用和web请求。每个受支持的安全对象类型都有自己的拦截器类，它是`AbstractSecurityInterceptor`的子类。重要的是，在调用`AbstractSecurityInterceptor`时，如果主体已经过身份验证，则`SecurityContextHolder`将包含有效的`Authentication`。

`AbstractSecurityInterceptor`为处理安全对象请求提供了一致的工作流程，通常：

1. 查找与当前请求关联的“配置属性”
2. 将安全对象，当前`Authentication`和配置属性提交到`AccessDecisionManager`以进行授权决策
3. （可选）更改发生调用的`Authentication`
4. 允许安全对象调用继续（假设已授予访问权限）
5. 调用返回后，调用`AfterInvocationManager`（如果已配置）。如果调用引发异常，则不会调用`AfterInvocationManager`。

### 2.配置属性

配置属性”可以被认为是对`AbstractSecurityInterceptor`使用的类具有特殊含义的String。它们由框架内的接口`ConfigAttribute`表示。

`AbstractSecurityInterceptor`配置了`SecurityMetadataSource`，用于查找安全对象的属性。通常，此配置将对用户隐藏。配置属性将作为安全方法的注释输入，或作为安全URL的访问属性输入。

- ##### RunAsManager

假设`AccessDecisionManager`决定允许请求，`AbstractSecurityInterceptor`通常只会继续请求。

- ##### AfterInvocationManager

在安全对象调用继续进行然后返回 - 这可能意味着方法调用完成或过滤器链继续进行 - `AbstractSecurityInterceptor`获得最后一次机会来处理调用。

## 四、核心服务

### 1.AuthenticationManager，ProviderManager和AuthenticationProvider

Spring Security中的默认实现称为`ProviderManager`，而不是处理身份验证请求本身。它会委托给已配置的`AuthenticationProvider`列表，每个列表都会被查询以查看它是否可以执行认证。每个提供程序将抛出异常或返回完全填充的`Authentication`对象。验证身份验证请求的最常用方法是加载相应的`UserDetails`并检查加载的密码与用户输入的密码。

身份验证机制（例如web表单登录处理过滤器）将注入`ProviderManager`的引用，并将调用它来处理其身份验证请求。

`ProviderManager`将尝试清除成功身份验证请求返回的`Authentication`对象中的任何敏感凭据信息。这可以防止密码保留的时间超过必要的时间。

当使用用户对象的缓存时，这可能会导致问题，例如，提高无状态应用程序的性能。如果`Authentication`包含对缓存中对象的引用（例如`UserDetails`实例）并且其凭据已删除，则将无法再对缓存的值进行身份验证。

Spring Security实现的最简单的`AuthenticationProvider`是`DaoAuthenticationProvider`，利用`UserDetailsService`（作为DAO）来查找用户名，密码和`GrantedAuthority`。它只需将`UsernamePasswordAuthenticationToken`中提交的密码与`UserDetailsService`加载的密码进行比较，即可对用户进行身份验证。

### 2.UserDetailsService实现



## 五、安全过滤器链

Spring Security在内部维护一个过滤器链，其中每个过滤器都有特定的责任，并根据所需的服务在配置中添加或删除过滤器。

### 1.DelegatingFilterProxy

在Spring Security中，过滤器类也是在应用程序上下文中定义的Spring bean，因此能够利用Spring丰富的依赖注入工具和生命周期接口。Spring的`DelegatingFilterProxy`提供了`web.xml`与应用程序上下文之间的链接。

### 2.FilterChainProxy

