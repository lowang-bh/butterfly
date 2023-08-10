# golang.md

- [golang.md](#golangmd)
  - [1. panic "extensions/extension.proto" is already registered](#1-panic-extensionsextensionproto-is-already-registered)

## 1. panic "extensions/extension.proto" is already registered

```sh
panic: proto: file "extensions/extension.proto" is already registered
    currently from:  "github.com/google/gnostic/extensions"
    previously from: "github.com/googleapis/gnostic/extensions"
```

搜索该panic，发现有一个类似问题<https://github.com/antrea-io/antrea/pull/4107>

排查过程：通过`go mod why`检查分配都是什么库依赖了这俩包，发现`k8s.io/kube-openapi/pkg/util/proto`包引用了和`k8s.io/client-go`不同的`gnostic`

```sh
# go mod why github.com/google/gnostic/extensions
k8s.io/client-go/tools/record
k8s.io/apimachinery/pkg/util/strategicpatch
k8s.io/kube-openapi/pkg/util/proto
github.com/google/gnostic/openapiv2
github.com/google/gnostic/compiler
github.com/google/gnostic/extensions

# go mod why github.com/googleapis/gnostic/extensions
k8s.io/client-go/kubernetes
k8s.io/client-go/discovery
github.com/googleapis/gnostic/OpenAPIv2
github.com/googleapis/gnostic/compiler
github.com/googleapis/gnostic/extensions
```

root cause: [https://github.com/kubernetes/kube-openapi/pull/285](https://github.com/kubernetes/kube-openapi/pull/285)开始，`github.com/google/gnostic`替换了原来的`github.com/googleapis/gnostic`库, 不同的库之间引用了不同版本的`gnostic`的库导致。

解决：`go.mod`里面修改`k8s.io/kube-openapi`版本到支持`github.com/googleapis`的版本号
