

---

# elasticsearch.service 的权限
1. 问题来源
    配置文件是 `/usr/lib/systemd/system/elasticsearch.service`：
    ``` service
    User=elasticsearch
    Group=elasticsearch
    ```
2. systemd相关的部分
    - [systemd.service(5)](https://man7.org/linux/man-pages/man5/systemd.service.5.html#SERVICE_TEMPLATES)
        打开`SERVICE TEMPLATES`
    - [SERVICE TEMPLATES](https://man7.org/linux/man-pages/man5/systemd.service.5.html#SERVICE_TEMPLATES)
        打开`USER/GROUP IDENTITY`
    - [USER/GROUP IDENTITY](https://man7.org/linux/man-pages/man5/systemd.exec.5.html#USER/GROUP_IDENTITY)
        ``` txt
            User=, Group=
                Set the UNIX user or group that the processes are executed
                as, respectively. Takes a single user or group name, or a
                numeric ID as argument. For system services (services run by
                the system service manager, i.e. managed by PID 1) and for
                user services of the root user (services managed by root's
                instance of systemd --user), the default is "root", but User=
                may be used to specify a different user. For user services of
                any other user, switching user identity is not permitted,
                hence the only valid setting is the same user the user's
                service manager is running as. If no group is set, the
                default group of the user is used. This setting does not
                affect commands whose command line is prefixed with "+".
        ```
        如上描述可知，`elasticsearch.service`中执行服务的user和group都是`elasticsearch`。
3. 


---
