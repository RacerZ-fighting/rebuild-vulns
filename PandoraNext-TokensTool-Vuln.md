### There is an Incorrect Access Control vulnerability in PandoraNext-TokensTool
### Version: <= v0.6.8 (latest version)
- github branch: main
### Description
- There is an authentication bypass vulnerability in PandoraNext-TokensTool. An attacker can exploit this vulnerability to access API without any token.
### Analysis
1. The affected source code class is `com.tokensTool.pandoraNext.interceptor.LoginCheckInterceptor`, and the affected function is `preHandle`. In the filter code, it uses `req.getRequestURL()` to obtain the request path,
   ![image](https://github.com/user-attachments/assets/4ca46847-5b4d-430b-a790-edb1bd76f5a0)
   and then determine whether the `url` contains `login`. If the condition is met, it will execute `return true` to bypass the Interceptor. Otherwise, it will continue to check if jwt token is correct.
2. The problem lies in using `req.getRequestURL()` to obtain the request path. The path obtained by this function will not parse special symbols, but will be passed on directly, so you can use `;` to bypass it.
Taking one of the backend interfaces `/api/selectSetting` as an example, using `/api/selectSetting;login` can make it bypass the `LoginCheckInterceptor` and access the system config without any token.
### Reproduce the vulnerablitity
Accessing `http://127.0.0.1:8081/api/selectSetting` directly will result in `NOT_LOGIN` response msg.
![image](https://github.com/user-attachments/assets/0cc472e4-bea5-4104-924b-06e145da6803)

However, accessing `http://127.0.0.1:8081/api/selectSetting;login` will bypass the authentication check and access the system configuation (content in config.json).
![image](https://github.com/user-attachments/assets/96defc39-c5a6-40e6-91d2-78aeea9bdb96)
