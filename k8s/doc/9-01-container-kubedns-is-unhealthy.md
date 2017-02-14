报错：container "kubedns" is unhealthy, it will be killed and re-created
---
解决方法：http://stackoverflow.com/questions/38451351/kubedns-container-creating-failed-with-the-skydns-rc-yaml-base-template-file

You must set a kube-master-url in args. For example:
``` shell
    # command = "/kube-dns"
    - --domain=fool.bar.
    - --dns-port=10053
    - --kube-master-url=http://8.9.6.4:8080
```

