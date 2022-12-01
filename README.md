# envory-filter-demo
envory-filter-demo

目前，k8s环境下的istio的envoy filter有两种实现方式，一种是wasm，支持多种语言编译成wasm的方式，以插件的形式注入到istio的过滤器链中，但目前测试下来，这种方式不太可靠，严重依赖对应的语言的sdk，比如golang的sdk支持就还处于测试阶段

当前目录下，envoy.yaml是envoy的配置，在使用envoy filter时用不到的，
仅用到了envoy-filter.yaml、main.go

下面是对应的步骤：

1. 编译成wasm
`tinygo build -o header-filter.wasm -scheduler=none -target=wasi`

2. 将wasm变成config map
`kubectl create cm header-filter-wasm --from-file=header-filter.wasm -n gcp`

3. 配置wasm的挂载信息
对应的deployment的istio注解加上：
`sidecar.istio.io/userVolume: '[{"name":"wasmfilters-dir","configMap": {"name": "header-filter-wasm"}}]'`
`sidecar.istio.io/userVolumeMount: '[{"mountPath":"/var/local/lib/wasm-filters","name":"wasmfilters-dir"}]'`

4. 部署envoy filter
`kube apply -f envoy-filter.yaml`

这种方式部署的envoy filter，一旦wasm有修改，都需要重启pod。

另一种是直接编写envoy filter，直接把lua脚本写在里面，这种修改会实时生效，对应的demo是header-filter.yaml。

另外，在envoy filter里，无法直接对外请求，只能为要请求的服务配置cluster，通过请求cluster做代理进行转发，详情可以看header-filter.yaml。

具体的lua脚本编写可参考: [envoy lua_filter]("https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/lua_filter")

注意：
要注意istio-proxy的日志等级，比如我的环境istio-proxy日志等级是warn，导致我打info一直出不来，还以为配置错了不生效。

笔者在使用go-wasm方案时，istio-proxy会报下面这种错，插件无法成功运行，导致istio-proxy无法运行，猜测是版本兼容问题（使用的版本是istio1.8，tinygo0.26，go1.19和1.18都是试过）暂时还没解决，

lua的那种方案倒是可以运行，没啥问题。

```log
2022-11-29T09:36:47.004533Z	error	envoy wasm	Wasm VM failed Failed to initialize Wasm code
2022-11-29T09:36:47.012533Z	warning	envoy config	gRPC config for type.googleapis.com/envoy.config.listener.v3.Listener rejected: Error adding/updating listener(s) virtualInbound: Unable to create Wasm HTTP filter
```

参考：

了解envoy里的基本概念:

[get-started-with-envoy-in-5-minutes](https://tetrate.io/blog/get-started-with-envoy-in-5-minutes/)
[service-mesh-102-envoy-configuration](https://www.cncf.io/blog/2021/11/09/service-mesh-102-envoy-configuration/)

istio envoy filter官方文档:

[istio envoy filter](https://istio.io/latest/docs/reference/config/networking/envoy-filter/)
[EnvoyFilter-Samples](https://github.com/istio/istio/wiki/EnvoyFilter-Samples)

使用go wasm 配置的envoy filter:

[基于assemblyscript/Go SDK开发Istio Envoy Wasm Filter](https://ieevee.com/tech/2021/07/23/wasm.html)
[基于 WASM 扩展 Envoy](https://juejin.cn/post/7064582996900184100)
[istio-wasm-extensions-and-ecosystem](https://tetrate.io/blog/istio-wasm-extensions-and-ecosystem/)
[wasm-modules-and-envoy-extensibility-explained-part-1](https://tetrate.io/blog/wasm-modules-and-envoy-extensibility-explained-part-1/)
[proxy-wasm-go-sdk README](https://github.com/tetratelabs/proxy-wasm-go-sdk/blob/main/README.md)
[proxy-wasm-go-sdk OVERVIEW](https://github.com/tetratelabs/proxy-wasm-go-sdk/blob/main/doc/OVERVIEW.md)
[proxy-wasm-go-sdk http header demo](https://github.com/tetratelabs/proxy-wasm-go-sdk/blob/main/examples/http_headers/README.md)
[envoy-wasm-failing-to-load-due-to-missing-import-using-net-http-go-module](https://stackoverflow.com/questions/70933142/envoy-wasm-failing-to-load-due-to-missing-import-using-net-http-go-module)

使用lua配置的envoy filter

[HTTP Request Body Logging with Istio & Envoy](https://devpress.csdn.net/k8s/62f4e7b57e668234661892be.html)
[Istio(十二)：Istio问题排查](https://www.cnblogs.com/renshengdezheli/p/16841748.html)
[Istio(七)：ServiceEntry，sidecar，Envoy Filter](https://www.cnblogs.com/renshengdezheli/p/16839960.html)