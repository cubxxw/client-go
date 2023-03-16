# client-go

👀 Some examples of client-go provide invocation methods.

+ [notion](https://nsddd.notion.site/Client-go-5d6f53fd968443b294c4af63f63de9d9)

github project address:

+ https://github.com/cubxxw/client-go

想要介绍 Kubernetes Client-Go, 那必须提前介绍 Kubernetes API

## Kubernetes API

Kubernetes API是一组REST API，用于与Kubernetes集群交互。这些API允许开发人员执行各种操作，包括管理Pod、Deployment、Service、Namespace等。Kubernetes API由一组资源对象表示，例如Pod、Service、ReplicaSet等。这些资源对象由Kubernetes API Server管理，并可以通过kubectl等工具进行查询和修改。

> Kubernetes API Server 提供的是默认的 HTTPS 服务，而且是双向的 TLS 认证，而我们目前的关注点是 API 本身，因此先通过 Kubectl 来代理 API Server 服务。

```go
kubectl proxy --port=8080
```

> 接下来就可以通过简单的 HTTP 请求来和 API Server 交互了：

```go
curl localhost:8080/version
```

我们可能还需要一个配置文件来描述 Deployment 资源，在本地创建一个 `nginx-deploy.yaml` 文件:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

要在 Kubernetes 集群中创建这个 Deployment 资源，可以使用 kubectl create 命令：

```
kubectl create -f nginx-deploy.yaml
```

可以使用 curl 命令通过 API Server 访问 Deployment 资源的相关信息，例如：

```
curl localhost:8080/apis/apps/v1/namespaces/default/deployments/nginx-deployment
```

要更新 Deployment 资源，可以使用 kubectl apply 命令：

```
kubectl apply -f nginx-deploy.yaml
```

要删除 Deployment 资源，可以使用 kubectl delete 命令：

```
kubectl delete deployment nginx-deployment
```

### 资源创建

Depolyment 的创建 API 是：`apps/v1` 中的 `Deployment`。

```yaml
POST /apis/apps/v1/namespaces/{namespace}/deployments
```

> github地址为：

[GitHub - kubernetes/client-go: Go client for Kubernetes.](https://github.com/kubernetes/client-go)

执行以下命令在 default 命名空间下创建一个 deployment：

```bash
curl -X POST --header 'Content-Type: application/yaml' --data-binary @nginx-deploy.yaml <http://localhost:8080/apis/apps/v1/namespaces/default/deployments>
```

### kubectl get —raw

`kubectl get --raw` 命令可以用来从 Kubernetes API 获取原始资源内容而不进行任何格式化或转换。这意味着该命令可以获取到 API 服务器返回的原始 JSON 或 YAML 格式的资源定义。此外，该命令还可以用来获取二进制文件，例如容器日志或 Kubernetes 对象的原始二进制数据。

使用 `kubectl get` 命令获取资源列表时，Kubernetes 会自动格式化输出，这有助于用户更好地理解资源。但是，在某些情况下，用户需要获取原始信息以进行高级操作。例如，用户可能需要将 Kubernetes 对象的 YAML 定义导出为文件，或者将容器日志保存到本地以进行分析。在这种情况下，`kubectl get --raw` 可以提供所需的原始信息。

需要注意的是，`kubectl get --raw` 命令返回的内容是未经处理的原始数据，因此必须小心使用，以避免意外地修改或删除 Kubernetes 对象。

## Client-go

Client-go是一个用于与Kubernetes API交互的Go库。它提供了广泛的功能，用于与Kubernetes API交互，包括强类型API、资源客户端、Watch API和动态客户端。使用client-go，开发人员可以轻松地在Kubernetes中创建、读取、更新和删除资源对象。

> 从这个`package`的名称来看，这应该是跟`k8s`打交道的客户端`client`的`go`实现，这一点没错，它定义了诸多资源的客户端`client`。

https://github.com/kubernetes/client-go

上面是 client-go 的 GitHub 仓库，不过这个库是 actions 以每天一次的频率从 Kubernetes/Kubernetes 主仓库中自动同步过来的。

> GitHub 中的位置是：kubernetes/stagin/src/k8s.io

client-go的版本规则是：MAJOR.MINOR.PATCH。其中：

+ MAJOR：向后不兼容的重大更改，例如API版本的更改。
+ MINOR：向后兼容的新功能，例如新的API资源。
+ PATCH：向后兼容的错误修复和性能改进。

例如，v0.19.0版本的client-go表示：

+ MAJOR版本为0，因此它是初始版本。
+ MINOR版本为19，表示它是第19个MINOR版本。
+ PATCH版本为0，表示它是初始版本。

当client-go的MAJOR版本发生更改时，新版本将不再与旧版本兼容，并且需要进行相应的更改。当client-go的MINOR版本发生更改时，新版本将包含新的功能，但不会破坏任何现有的功能。当client-go的PATCH版本发生更改时，新版本将包含修复错误和性能改进，不会破坏任何现有的功能。

| Client-go 版本 | Kubernetes 版本 |
| -------------- | --------------- |
| v0.16.0        | v1.6            |
| v0.17.0        | v1.7            |
| v0.18.0        | v1.8            |
| v0.19.0        | v1.9            |
| v0.20.0        | v1.10           |
| v0.21.0        | v1.11           |
| v0.22.0        | v1.12           |
| v0.23.0        | v1.13           |
| v0.24.0        | v1.14           |
| v0.25.0        | v1.15           |
| v0.26.0        | v1.16           |
| v0.27.0        | v1.17           |
| v0.28.0        | v1.18           |
| v0.29.0        | v1.19           |

## Client-go目录结构

以下是client-go库的主要目录和文件：

+ `/discovery`：发现和获取Kubernetes API资源的代码。
+ `/dynamic`：动态客户端库，用于与Kubernetes API交互，而无需为每个资源生成代码。
+ `/informers`：用于监视Kubernetes资源变化的代码。
+ `/listers`：用于从Kubernetes服务器获取资源列表的代码。
+ `/rest`：用于与Kubernetes API交互的代码。
+ `/scale`：用于与Kubernetes资源的自动缩放相关的代码。
+ `/tools`：用于测试和其他实用程序的代码。
+ `/util`：用于客户端库的辅助功能的代码。

以下是一些重要的文件：

+ `clientset.go`：客户端库的主要入口点。
+ `config.go`：用于配置客户端库的代码。
+ `discovery.go`：用于发现和获取Kubernetes API资源的代码。
+ `rest.go`：用于与Kubernetes API交互的代码。

## 目录结构解释

+ `/discovery`：该目录包含用于发现和获取Kubernetes API资源的代码。这些资源包括Pod、Service、ReplicationController等。`discovery`目录中的代码可以帮助开发人员发现和使用这些资源。
+ `/dynamic`：该目录包含动态客户端库，用于与Kubernetes API交互，而无需生成代码。这对于构建需要与任意Kubernetes资源交互的通用工具和实用程序非常有用。
+ `/informers`：该目录包含用于监视Kubernetes资源变化的代码。这些变化可以包括资源的创建、更新和删除。`informers`目录中的代码可以帮助开发人员构建控制器和其他需要对Kubernetes环境中的变化做出反应的应用程序。
+ `/listers`：该目录包含用于从Kubernetes服务器获取资源列表的代码。这些资源列表包括Pod、Service、Namespace等。`listers`目录中的代码可以帮助开发人员更轻松地获取有关Kubernetes资源的信息。
+ `/rest`：该目录包含用于与Kubernetes API交互的代码。这些API包括Pod、Service、Namespace等。`rest`目录中的代码可以帮助开发人员执行各种操作，包括管理Pod、Deployment、Service、Namespace等。
+ `/scale`：该目录包含用于与Kubernetes资源的自动缩放相关的代码。这些资源包括Deployment、ReplicaSet、StatefulSet等。`scale`目录中的代码可以帮助开发人员自动缩放与Kubernetes资源相关的组件。
+ `/tools`：该目录包含用于测试和其他实用程序的代码。这些实用程序包括代码生成器、测试工具等。`tools`目录中的代码可以帮助开发人员更轻松地测试和使用client-go库。
+ `/util`：该目录包含用于客户端库的辅助功能的代码。这些功能包括对Kubernetes API对象的类型转换、对象比较等。`util`目录中的代码可以帮助开发人员更轻松地使用client-go库。

这么多功能，我们怎么使用呢？其实官方已经给了很多案例了（参考`client-go/examples`）。

+ client-go/examples

  官方提供了很多使用client-go的例子，这些例子可以在`client-go/examples`目录中找到。以下是一些例子：

  + `create-update-delete-deployment`：演示如何创建、更新和删除Deployment资源。这个例子还演示了如何使用Informer和Lister来监视和获取Deployment资源。
  + `cronjob`：演示如何使用client-go创建和管理CronJob资源。
  + `custom-resource-definition`：演示如何使用client-go创建和管理自定义资源。
  + `daemonset`：演示如何使用client-go创建和管理DaemonSet资源。
  + `job`：演示如何使用client-go创建和管理Job资源。
  + `pod-creation-deletion-grace-period`：演示了Pod的创建、删除和优雅的停机期（grace period）。
  + `portforward`：演示如何使用client-go进行端口转发，从而可以直接访问Kubernetes Pod中运行的应用程序。
  + `rolling-update-deployment`：演示如何使用client-go执行滚动更新（rolling update）操作，以便更改Deployment资源的Pod模板。
  + `service`：演示如何使用client-go创建和管理Service资源。

  这些示例提供了一个很好的起点，从中开发人员可以了解如何使用client-go与Kubernetes API交互。

### client-go examplex

获取client-go的方式有两种：

1.从GitHub上下载源代码。

您可以从 Kubernetes 的 GitHub 仓库中克隆 client-go 的最新源代码，然后将其构建为可执行文件。要获取源代码，请使用以下命令：

```
git clone <https://github.com/kubernetes/client-go.git>
```

1. 使用go get命令获取client-go。

使用以下命令可以直接从 GitHub 获取 client-go：

```
go get k8s.io/client-go/...
```

这将在您的 GOPATH 中安装最新版本的 client-go。

⚠️ 请注意，使用 go get 命令安装的 client-go 版本可能与 Kubernetes 版本不兼容，因此请确保使用正确的版本。建议使用您的 Kubernetes 发行版中提供的 client-go 版本。

如果您想要使用特定版本的 client-go，请使用以下命令：

```
go get k8s.io/client-go@v0.19.0
```

这将安装 client-go v0.19.0 版本。请注意，在使用 go get 命令时，必须使用 `@` 符号指定版本号。

### README

这个仓库是用来学习 client-go 的，和之前的 go-mod 仓库一样，当然也可接入：

`client-go.go` 文件：

```bash
package main

import (
	"flag"
	"fmt"
	"os"
	"path/filepath"
	"time"

	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/client-go/kubernetes"
	"k8s.io/client-go/tools/clientcmd"
)

func main() {
	var kubeconfig *string
	if home := homeDir(); home != "" {
		kubeconfig = flag.String("kubeconfig", filepath.Join(home, ".kube", "config"), "(optional) absolute path to the kubeconfig file")
	} else {
		kubeconfig = flag.String("kubeconfig", "", "absolute path to the kubeconfig file")
	}
	flag.Parse()
	// uses the current context in kubeconfig
	config, err := clientcmd.BuildConfigFromFlags("", *kubeconfig)
	if err != nil {
		panic(err.Error())
	}
	// creates the clientset
	clientset, err := kubernetes.NewForConfig(config)
	if err != nil {
		panic(err.Error())
	}
	for {
		pods, err := clientset.CoreV1().Pods("").List(metav1.ListOptions{})
		if err != nil {
			panic(err.Error())
		}
		fmt.Printf("There are %d pods in the cluster\\n", len(pods.Items))
		time.Sleep(10 * time.Second)
	}
}

func homeDir() string {
	if h := os.Getenv("HOME"); h != "" {
		return h
	}
	return os.Getenv("USERPROFILE") // windows
}
```

这个程序是一个示例，它使用`client-go`连接到Kubernetes集群，并列出所有Pod的数量。以下是程序的一些关键部分：

1. 通过读取命令行参数来获取`kubeconfig`文件的路径，该文件用于配置连接到Kubernetes集群的客户端。
2. 使用`client-go`的`BuildConfigFromFlags`函数来获取配置。
3. 使用`client-go`的`kubernetes.NewForConfig`函数创建一个`clientset`，用于与Kubernetes API交互。
4. 使用`clientset`的`CoreV1().Pods().List()`函数获取所有Pod的列表，并使用`len()`函数计算Pod的数量。
5. 在一个循环中运行程序，每10秒钟打印一次Pod的数量。

如果发现报错，修改为以下代码：

```go
// pods, err := clientset.CoreV1().Pods("").List(metav1.ListOptions{})
pods, err := clientset.CoreV1().Pods("").List(context.Background(), metav1.ListOptions{})
```

PodInterface：

```go
// PodInterface has methods to work with Pod resources.
type PodInterface interface {
	Create(ctx context.Context, pod *v1.Pod, opts metav1.CreateOptions) (*v1.Pod, error)
	Update(ctx context.Context, pod *v1.Pod, opts metav1.UpdateOptions) (*v1.Pod, error)
	UpdateStatus(ctx context.Context, pod *v1.Pod, opts metav1.UpdateOptions) (*v1.Pod, error)
	Delete(ctx context.Context, name string, opts metav1.DeleteOptions) error
	DeleteCollection(ctx context.Context, opts metav1.DeleteOptions, listOpts metav1.ListOptions) error
	Get(ctx context.Context, name string, opts metav1.GetOptions) (*v1.Pod, error)
	List(ctx context.Context, opts metav1.ListOptions) (*v1.PodList, error)
	Watch(ctx context.Context, opts metav1.ListOptions) (watch.Interface, error)
	Patch(ctx context.Context, name string, pt types.PatchType, data []byte, opts metav1.PatchOptions, subresources ...string) (result *v1.Pod, err error)
	Apply(ctx context.Context, pod *corev1.PodApplyConfiguration, opts metav1.ApplyOptions) (result *v1.Pod, err error)
	ApplyStatus(ctx context.Context, pod *corev1.PodApplyConfiguration, opts metav1.ApplyOptions) (result *v1.Pod, err error)
	UpdateEphemeralContainers(ctx context.Context, podName string, pod *v1.Pod, opts metav1.UpdateOptions) (*v1.Pod, error)

	PodExpansion
}
```

这是一个关于Kubernetes的API接口文档，其中包含了多个API接口方法。以下是每个方法的简要解释：

+ 创建Pod资源：创建一个新的Pod，并指定所需的容器和其它配置。
+ 更新Pod资源：更新一个已经存在的Pod，例如更新容器的镜像版本，或者增加/删除容器等。
+ 删除Pod资源：删除一个已经存在的Pod，包括其下的所有容器。
+ 获取Pod列表：获取集群中所有Pod的列表。
+ 获取单个Pod资源：获取指定名称的Pod资源的详细信息。
+ 打补丁：在不删除Pod的情况下更新它的属性，例如容器的镜像版本或者环境变量等。
+ 更新临时容器：在Pod中创建一个临时容器，在其中执行一些命令，并在完成后将其删除。

这些方法是用于与Kubernetes API交互的客户端库的一部分，例如client-go。这个API接口文档还提供了关于如何配置连接到Kubernetes集群的客户端的详细信息。包括如何获取Kubernetes的配置文件kubeconfig的绝对路径，以及如何使用rest.config文件配置client-go连接到Kubernetes API服务器。此外，还提供了如何解决认证问题、如何调用discovery和dynamic API等其他相关内容。

总之，本文档提供了与Kubernetes API交互的基础知识和相关配置信息，对于使用Kubernetes进行应用程序开发和管理的人员来说是非常有用的。

***\**\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*运行结果：\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\****

```bash
❯ go run client-go.go
There are 35 pods in the cluster
There are 35 pods in the cluster
There are 35 pods in the cluster
There are 35 pods in the cluster
There are 35 pods in the cluster
There are 35 pods in the cluster
```

这个程序使用`client-go`连接到Kubernetes集群，并列出所有Pod的数量。在程序中，使用`client-go`的`BuildConfigFromFlags`函数来获取配置，使用`client-go`的`kubernetes.NewForConfig`函数创建一个`clientset`，用于与Kubernetes API交互。同时，使用`clientset`的`CoreV1().Pods().List()`函数获取所有Pod的列表，并使用`len()`函数计算Pod的数量。在一个循环中运行程序，每10秒钟打印一次Pod的数量。因此，运行结果是每10秒一次的输出，显示当前集群中有多少个Pod。

可以使用以下命令将 `kubectl get pod -A` 命令的输出通过管道传送给 `wc` 命令，以统计有多少个 Pod：

```
❯ kubectl get pod -A | wc -l
36
```

> 默认是 36 - 1

## 源码分析

### ***\**\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*kubeconfig:\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\**\***

```bash
kubeconfig = flag.String("kubeconfig", filepath.Join(home, ".kube", "config"), "(optional) absolute path to the kubeconfig file")
```

获取kubernetes配置文件kubeconfig的绝对路径。一般路径为`$HOME/.kube/config`。该文件主要用来配置本地连接的kubernetes集群。

### **config内容如下：**

```yaml
apiVersion: v1
clusters:
- cluster:
    server: http://<kube-master-ip>:8080
  name: k8s
contexts:
- context:
    cluster: k8s
    namespace: default
    user: ""
  name: default
current-context: default
kind: Config
preferences: {}
users: []
```

`config`文件是用于配置连接到Kubernetes集群的客户端的文件。文件分为以下几个字段：

+ `apiVersion`：Kubernetes API的版本号。
+ `kind`：资源类型，这里是`Config`。
+ `clusters`：定义集群信息。
+ `users`：定义用户信息。
+ `contexts`：定义上下文信息，即哪个用户使用哪个集群。
+ `current-context`：当前使用的上下文。

在`clusters`字段中，每个集群都有一个`name`和一个`cluster`。`name`用于标识集群，`cluster`包含以下信息：

+ `server`：Kubernetes API服务器的地址。
+ `certificate-authority`：用于验证API服务器证书的CA证书文件路径。
+ `insecure-skip-tls-verify`：是否跳过验证API服务器证书。默认为`false`。

在`users`字段中，每个用户都有一个`name`和一个`user`。`name`用于标识用户，`user`包含以下信息：

+ `client-certificate`：用于验证客户端证书的证书文件路径。
+ `client-key`：用于验证客户端证书的私钥文件路径。
+ `username`：用于基本身份验证的用户名。
+ `password`：用于基本身份验证的密码。

在`contexts`字段中，每个上下文都有一个`name`和一个`context`。`name`用于标识上下文，`context`包含以下信息：

+ `cluster`：使用的集群。
+ `user`：使用的用户。
+ `namespace`：使用的命名空间。

在`current-context`字段中，指定当前使用的上下文的名称。

这些字段共同构成了一个完整的`Config`文件，用于配置连接到Kubernetes集群的客户端。

### ***\**\*rest.config：\*\**\***

`rest.config`是一个用于配置`client-go`连接到Kubernetes API服务器的配置文件。它可以从`kubeconfig`文件中创建，也可以直接在代码中创建。下面是一个例子：

```go
import (
	"k8s.io/client-go/rest"
	"k8s.io/client-go/tools/clientcmd"
)

func main() {
	// create a config from the kubeconfig file
	config, err := clientcmd.BuildConfigFromFlags("", "/path/to/kubeconfig")
	if err != nil {
		panic(err.Error())
	}

	// create a new clientset using the config
	clientset, err := kubernetes.NewForConfig(config)
	if err != nil {
		panic(err.Error())
	}

	// use the clientset to interact with the Kubernetes API
	...
}
```

`rest.config`包含以下字段：

+ `Host`：Kubernetes API服务器的地址。
+ `APIPath`：Kubernetes API的路径。默认为`/api`。
+ `ContentConfig`：用于序列化和反序列化Kubernetes对象的配置。
+ `Username`：用于基本身份验证的用户名。
+ `Password`：用于基本身份验证的密码。
+ `BearerToken`：用于Bearer令牌身份验证的令牌。
+ `BearerTokenFile`：包含Bearer令牌的文件路径。
+ `Impersonate`：用于模拟用户身份的配置。
+ `TLSClientConfig`：用于与Kubernetes API服务器进行TLS通信的配置。
+ `UserAgent`：用于标识客户端的用户代理。

这些字段可以在代码中进行设置，以便`client-go`能够连接到正确的Kubernetes API服务器。

通过参数（master的url或者kubeconfig路径）和`BuildConfigFromFlags`方法来获取`rest.Config`对象，一般是通过参数kubeconfig的路径。

```yaml
config, err := clientcmd.BuildConfigFromFlags("", *kubeconfig)
```

### ***\*clientset:\****

程序使用`client-go`连接到Kubernetes集群，并列出所有Pod的数量。在程序中，使用`client-go`的`BuildConfigFromFlags`函数来获取配置，使用`client-go`的`kubernetes.NewForConfig`函数创建一个`clientset`，用于与Kubernetes API交互。同时，使用`clientset`的`CoreV1().Pods().List()`函数获取所有Pod的列表，并使用`len()`函数计算Pod的数量。在一个循环中运行程序，每10秒钟打印一次Pod的数量。如果遇到错误，需要修改代码并使用`context.Background()`函数来处理。此外，Kubernetes的API接口文档提供了与Kubernetes API交互的基础知识和相关配置信息，对于使用Kubernetes进行应用程序开发和管理的人员来说是非常有用的。其中，`PodInterface`包含了多个API接口方法，例如创建、更新、删除、获取、打补丁等。

通过`*rest.Config`参数和`NewForConfig`方法来获取`clientset`对象，`clientset`是多个`client`的集合，每个`client`可能包含不同版本的方法调用。

```go
func kubernetes.NewForConfig(c *rest.Config) (*kubernetes.Clientset, error)
```

💡简单的一个案例如下：

```go
// creates the clientset
	clientset, err := kubernetes.NewForConfig(config)
	if err != nil {
		panic(err.Error())
	}
	for {
		pods, err := clientset.CoreV1().Pods("").List(context.Background(), metav1.ListOptions{})
		if err != nil {
			panic(err.Error())
		}
		fmt.Printf("There are %d pods in the cluster\\n", len(pods.Items))
		time.Sleep(10 * time.Second)
	}
```

### ***\*NewForConfig\****

`NewForConfig`函数就是初始化clientset中的每个client。

`NewForConfig`是client-go库中的一个方法，用于根据rest.Config对象创建一个新的Kubernetes客户端。该方法返回一个指向`kubernetes.Clientset`对象的指针，后者是另一个client-go库中的类型。 `Clientset`封装了与Kubernetes API进行交互的所有方法，例如获取Pod列表，创建新的Deployment等等。

`NewForConfig`方法的定义如下：

```go
func NewForConfig(c *rest.Config) (*Clientset, error) {
    configShallowCopy := *c
    ...
    var cs Clientset
    cs.appsV1beta1, err = appsv1beta1.NewForConfig(&configShallowCopy)
    ...
    cs.coreV1, err = corev1.NewForConfig(&configShallowCopy)
    ...
}
```

其中参数`c`是一个指向rest.Config对象的指针，该对象描述了与Kubernetes API服务器进行通信所需的配置。`NewForConfig`方法返回一个指向`kubernetes.Clientset`对象的指针及一个可选的错误。如果在创建客户端时出现错误，则返回非零错误值。

以下是一个使用`NewForConfig`方法的示例：

```go
import (
    "k8s.io/client-go/kubernetes"
    "k8s.io/client-go/rest"
)

func main() {
    // Create a new Kubernetes client with the given config
    config, err := rest.InClusterConfig()
    if err != nil {
        panic(err)
    }
    clientset, err := kubernetes.NewForConfig(config)
    if err != nil {
        panic(err)
    }

    // Use the clientset to interact with the Kubernetes API
    ...
}
```

在此示例中，`rest.InClusterConfig`方法用于获取当前运行的Pod的Kubernetes配置。然后，使用`NewForConfig`方法创建一个新的客户端，并将其存储在名为`clientset`的变量中。这个客户端可以用来与Kubernetes API服务器进行交互，例如获取Pod列表或创建新的Deployment。

总之，`NewForConfig`方法是client-go库中的一个重要方法，用于创建新的Kubernetes客户端。

## ***\*clientset的结构体\****



```go
// Clientset contains the clients for groups. Each group has exactly one
// version included in a Clientset.
type Clientset struct {
    *discovery.DiscoveryClient
    admissionregistrationV1alpha1 *admissionregistrationv1alpha1.AdmissionregistrationV1alpha1Client
    appsV1beta1                   *appsv1beta1.AppsV1beta1Client
    appsV1beta2                   *appsv1beta2.AppsV1beta2Client
    authenticationV1              *authenticationv1.AuthenticationV1Client
    authenticationV1beta1         *authenticationv1beta1.AuthenticationV1beta1Client
    authorizationV1               *authorizationv1.AuthorizationV1Client
    authorizationV1beta1          *authorizationv1beta1.AuthorizationV1beta1Client
    autoscalingV1                 *autoscalingv1.AutoscalingV1Client
    autoscalingV2beta1            *autoscalingv2beta1.AutoscalingV2beta1Client
    batchV1                       *batchv1.BatchV1Client
    batchV1beta1                  *batchv1beta1.BatchV1beta1Client
    batchV2alpha1                 *batchv2alpha1.BatchV2alpha1Client
    certificatesV1beta1           *certificatesv1beta1.CertificatesV1beta1Client
    coreV1                        *corev1.CoreV1Client
    extensionsV1beta1             *extensionsv1beta1.ExtensionsV1beta1Client
    networkingV1                  *networkingv1.NetworkingV1Client
    policyV1beta1                 *policyv1beta1.PolicyV1beta1Client
    rbacV1                        *rbacv1.RbacV1Client
    rbacV1beta1                   *rbacv1beta1.RbacV1beta1Client
    rbacV1alpha1                  *rbacv1alpha1.RbacV1alpha1Client
    schedulingV1alpha1            *schedulingv1alpha1.SchedulingV1alpha1Client
    settingsV1alpha1              *settingsv1alpha1.SettingsV1alpha1Client
    storageV1beta1                *storagev1beta1.StorageV1beta1Client
    storageV1                     *storagev1.StorageV1Client
}
```

上面的结构体是`clientset`，包含了不同版本的API集合。其中，每个字段都对应了一个API组，例如`coreV1`对应了`core`组。每个API组都包含了一组API接口，例如`coreV1`包含了与Pod、Service、Namespace等资源相关的API接口。通过`clientset`的相应字段，可以访问Kubernetes API提供的不同版本的API接口。

例如，要获取Pod资源的API接口，可以使用`clientset`的`CoreV1().Pods()`方法，该方法返回一个`PodInterface`对象。`PodInterface`包含了多个API接口方法，例如创建、更新、删除、获取、打补丁等。

> 我们上面使用的是 `pods, err := clientset.CoreV1().Pods("").List(context.Background(), metav1.ListOptions{})`

此外，`clientset`还包含了一个`discovery.DiscoveryClient`字段，用于发现Kubernetes API服务器上可用的API资源。可以使用`discovery.DiscoveryClient`的`ServerGroups()`方法获取服务器上可用的API组信息，使用`discovery.DiscoveryClient`的`ServerResourcesForGroupVersion()`方法获取特定API组版本的API资源信息。

总之，`clientset`是`client-go`库中的一个重要结构体，用于与Kubernetes API进行交互。每个`clientset`字段都对应了一个API组，包含了一组API接口。可以使用`clientset`来与Kubernetes API进行交互，例如获取Pod列表或创建新的Deployment。

### ***\*clientset.Interface\****

clientset实现了以下的Interface，因此可以通过调用以下方法获得具体的client。

```go
pods, err := clientset.CoreV1().Pods("").List(metav1.ListOptions{})
```

例如：

+ `CoreV1()`：获取CoreV1接口。
+ `AppsV1beta1()`：获取AppsV1beta1接口。
+ `AppsV1beta2()`：获取AppsV1beta2接口。
+ `AuthenticationV1()`：获取AuthenticationV1接口。
+ `AuthorizationV1()`：获取AuthorizationV1接口。
+ `BatchV1()`：获取BatchV1接口。
+ `BatchV1beta1()`：获取BatchV1beta1接口。
+ `CertificatesV1beta1()`：获取CertificatesV1beta1接口。
+ `NetworkingV1()`：获取NetworkingV1接口。
+ `PolicyV1beta1()`：获取PolicyV1beta1接口。
+ `RbacV1()`：获取RbacV1接口。
+ `SchedulingV1alpha1()`：获取SchedulingV1alpha1接口。
+ `StorageV1beta1()`：获取StorageV1beta1接口。
+ `StorageV1()`：获取StorageV1接口。

可以使用这些接口来与Kubernetes API进行交互，例如获取Pod列表或创建新的Deployment。

### **clientset的方法集接口**

+ https://github.com/kubernetes/client-go/blob/87887458218a51f3944b2f4c553eb38173458e97/kubernetes/clientset.go#L54

```go
type Interface interface {
	Discovery() discovery.DiscoveryInterface
	AdmissionregistrationV1alpha1() admissionregistrationv1alpha1.AdmissionregistrationV1alpha1Interface
	AdmissionregistrationV1beta1() admissionregistrationv1beta1.AdmissionregistrationV1beta1Interface
	// Deprecated: please explicitly pick a version if possible.
	Admissionregistration() admissionregistrationv1beta1.AdmissionregistrationV1beta1Interface
	AppsV1beta1() appsv1beta1.AppsV1beta1Interface
	AppsV1beta2() appsv1beta2.AppsV1beta2Interface
	AppsV1() appsv1.AppsV1Interface
	// Deprecated: please explicitly pick a version if possible.
	Apps() appsv1.AppsV1Interface
	AuthenticationV1() authenticationv1.AuthenticationV1Interface
	// Deprecated: please explicitly pick a version if possible.
	Authentication() authenticationv1.AuthenticationV1Interface
	AuthenticationV1beta1() authenticationv1beta1.AuthenticationV1beta1Interface
	AuthorizationV1() authorizationv1.AuthorizationV1Interface
	// Deprecated: please explicitly pick a version if possible.
	Authorization() authorizationv1.AuthorizationV1Interface
	AuthorizationV1beta1() authorizationv1beta1.AuthorizationV1beta1Interface
	AutoscalingV1() autoscalingv1.AutoscalingV1Interface
	// Deprecated: please explicitly pick a version if possible.
	Autoscaling() autoscalingv1.AutoscalingV1Interface
	AutoscalingV2beta1() autoscalingv2beta1.AutoscalingV2beta1Interface
	BatchV1() batchv1.BatchV1Interface
	// Deprecated: please explicitly pick a version if possible.
	Batch() batchv1.BatchV1Interface
	BatchV1beta1() batchv1beta1.BatchV1beta1Interface
	BatchV2alpha1() batchv2alpha1.BatchV2alpha1Interface
	CertificatesV1beta1() certificatesv1beta1.CertificatesV1beta1Interface
	// Deprecated: please explicitly pick a version if possible.
	Certificates() certificatesv1beta1.CertificatesV1beta1Interface
	CoreV1() corev1.CoreV1Interface
	// Deprecated: please explicitly pick a version if possible.
	Core() corev1.CoreV1Interface
	EventsV1beta1() eventsv1beta1.EventsV1beta1Interface
	// Deprecated: please explicitly pick a version if possible.
	Events() eventsv1beta1.EventsV1beta1Interface
	ExtensionsV1beta1() extensionsv1beta1.ExtensionsV1beta1Interface
	// Deprecated: please explicitly pick a version if possible.
	Extensions() extensionsv1beta1.ExtensionsV1beta1Interface
	NetworkingV1() networkingv1.NetworkingV1Interface
	// Deprecated: please explicitly pick a version if possible.
	Networking() networkingv1.NetworkingV1Interface
	PolicyV1beta1() policyv1beta1.PolicyV1beta1Interface
	// Deprecated: please explicitly pick a version if possible.
	Policy() policyv1beta1.PolicyV1beta1Interface
	RbacV1() rbacv1.RbacV1Interface
	// Deprecated: please explicitly pick a version if possible.
	Rbac() rbacv1.RbacV1Interface
	RbacV1beta1() rbacv1beta1.RbacV1beta1Interface
	RbacV1alpha1() rbacv1alpha1.RbacV1alpha1Interface
	SchedulingV1alpha1() schedulingv1alpha1.SchedulingV1alpha1Interface
	// Deprecated: please explicitly pick a version if possible.
	Scheduling() schedulingv1alpha1.SchedulingV1alpha1Interface
	SettingsV1alpha1() settingsv1alpha1.SettingsV1alpha1Interface
	// Deprecated: please explicitly pick a version if possible.
	Settings() settingsv1alpha1.SettingsV1alpha1Interface
	StorageV1beta1() storagev1beta1.StorageV1beta1Interface
	StorageV1() storagev1.StorageV1Interface
	// Deprecated: please explicitly pick a version if possible.
	Storage() storagev1.StorageV1Interface
	StorageV1alpha1() storagev1alpha1.StorageV1alpha1Interface
}
```

上述接口方法是clientset中的方法集接口。这些接口方法分别返回与Kubernetes API进行交互的不同版本的API接口。例如，如果要获取Pod资源的API接口，则可以使用`clientset`的`CoreV1().Pods()`方法，该方法返回一个`PodInterface`对象。`PodInterface`包含了多个API接口方法，例如创建、更新、删除、获取、打补丁等。

+ `Discovery()`：获取发现接口。
+ `AdmissionregistrationV1alpha1()`：获取AdmissionregistrationV1alpha1接口。
+ `AdmissionregistrationV1beta1()`：获取AdmissionregistrationV1beta1接口。
+ `AppsV1beta1()`：获取AppsV1beta1接口。
+ `AppsV1beta2()`：获取AppsV1beta2接口。
+ `AppsV1()`：获取AppsV1接口。
+ `AuthenticationV1()`：获取AuthenticationV1接口。
+ `AuthenticationV1beta1()`：获取AuthenticationV1beta1接口。
+ `AuthorizationV1()`：获取AuthorizationV1接口。
+ `AuthorizationV1beta1()`：获取AuthorizationV1beta1接口。
+ `AutoscalingV1()`：获取AutoscalingV1接口。
+ `AutoscalingV2beta1()`：获取AutoscalingV2beta1接口。
+ `BatchV1()`：获取BatchV1接口。
+ `BatchV1beta1()`：获取BatchV1beta1接口。
+ `BatchV2alpha1()`：获取BatchV2alpha1接口。
+ `CertificatesV1beta1()`：获取CertificatesV1beta1接口。
+ `CoreV1()`：获取CoreV1接口。
+ `EventsV1beta1()`：获取EventsV1beta1接口。
+ `ExtensionsV1beta1()`：获取ExtensionsV1beta1接口。
+ `NetworkingV1()`：获取NetworkingV1接口。
+ `PolicyV1beta1()`：获取PolicyV1beta1接口。
+ `RbacV1()`：获取RbacV1接口。
+ `RbacV1beta1()`：获取RbacV1beta1接口。
+ `RbacV1alpha1()`：获取RbacV1alpha1接口。
+ `SchedulingV1alpha1()`：获取SchedulingV1alpha1接口。
+ `SettingsV1alpha1()`：获取SettingsV1alpha1接口。
+ `StorageV1beta1()`：获取StorageV1beta1接口。
+ `StorageV1()`：获取StorageV1接口。
+ `StorageV1alpha1()`：获取StorageV1alpha1接口。

每个接口方法返回一个特定版本的API接口，用于与Kubernetes API进行交互。

### ***\*CoreV1Client\****

我们以clientset中的`CoreV1Client`为例做分析。

通过传入的配置信息`rest.Config`初始化`CoreV1Client`对象

+ https://github.com/kubernetes/client-go/blob/87887458218a51f3944b2f4c553eb38173458e97/kubernetes/clientset.go#L464

### corev1.NewForConfig

+ [k8s.io/client-go/kubernetes/typed/core/v1/core_client.go](https://github.com/kubernetes/client-go/blob/87887458218a51f3944b2f4c553eb38173458e97/kubernetes/typed/core/v1/core_client.go#L116:6)

```go
// NewForConfig creates a new CoreV1Client for the given config.
func NewForConfig(c *rest.Config) (*CoreV1Client, error) {
	config := *c
	if err := setConfigDefaults(&config); err != nil {
		return nil, err
	}
	client, err := rest.RESTClientFor(&config)
	if err != nil {
		return nil, err
	}
	return &CoreV1Client{client}, nil
}
```

`corev1.NewForConfig`方法本质是调用了`rest.RESTClientFor(&config)`方法创建`RESTClient`对象，即`CoreV1Client`的本质就是一个`RESTClient`对象。

### ***\*CoreV1Client结构体\****

以下是`CoreV1Client`结构体的定义：

+ [k8s.io/client-go/kubernetes/typed/core/v1/core_client.go](https://github.com/kubernetes/client-go/blob/87887458218a51f3944b2f4c553eb38173458e97/kubernetes/typed/core/v1/core_client.go#L47:6)

```go
// CoreV1Client is used to interact with features provided by the  group.
type CoreV1Client struct {
    restClient rest.Interface
}
```

`CoreV1Client` 实现了`CoreV1Interface` 的接口，即以下方法，从而对kubernetes的资源对象进行增删改查的操作。

+ [k8s.io/client-go/kubernetes/typed/core/v1/core_client.go](https://github.com/kubernetes/client-go/blob/87887458218a51f3944b2f4c553eb38173458e97/kubernetes/typed/core/v1/core_client.go#L51)

```go
//CoreV1Client的方法
func (c *CoreV1Client) ComponentStatuses() ComponentStatusInterface {...}
//ConfigMaps
func (c *CoreV1Client) ConfigMaps(namespace string) ConfigMapInterface {...}
//Endpoints
func (c *CoreV1Client) Endpoints(namespace string) EndpointsInterface {...}
func (c *CoreV1Client) Events(namespace string) EventInterface {...}
func (c *CoreV1Client) LimitRanges(namespace string) LimitRangeInterface {...}
//Namespaces
func (c *CoreV1Client) Namespaces() NamespaceInterface {...}
//Nodes
func (c *CoreV1Client) Nodes() NodeInterface {...}
func (c *CoreV1Client) PersistentVolumes() PersistentVolumeInterface {...}
func (c *CoreV1Client) PersistentVolumeClaims(namespace string) PersistentVolumeClaimInterface {...}
//Pods
func (c *CoreV1Client) Pods(namespace string) PodInterface {...}
func (c *CoreV1Client) PodTemplates(namespace string) PodTemplateInterface {...}
//ReplicationControllers
func (c *CoreV1Client) ReplicationControllers(namespace string) ReplicationControllerInterface {...}
func (c *CoreV1Client) ResourceQuotas(namespace string) ResourceQuotaInterface {...}
func (c *CoreV1Client) Secrets(namespace string) SecretInterface {...}
//Services
func (c *CoreV1Client) Services(namespace string) ServiceInterface {...}
func (c *CoreV1Client) ServiceAccounts(namespace string) ServiceAccountInterface {...}
```

### ***\*CoreV1Interface\****

+ [k8s.io/client-go/kubernetes/typed/core/v1/core_client.go](https://github.com/kubernetes/client-go/blob/87887458218a51f3944b2f4c553eb38173458e97/kubernetes/typed/core/v1/core_client.go#L26)

```go
type CoreV1Interface interface {
    RESTClient() rest.Interface
    ComponentStatusesGetter
    ConfigMapsGetter
    EndpointsGetter
    EventsGetter
    LimitRangesGetter
    NamespacesGetter
    NodesGetter
    PersistentVolumesGetter
    PersistentVolumeClaimsGetter
    PodsGetter
    PodTemplatesGetter
    ReplicationControllersGetter
    ResourceQuotasGetter
    SecretsGetter
    ServicesGetter
    ServiceAccountsGetter
}
```

`CoreV1Interface`中包含了各种`kubernetes`对象的调用接口，例如`PodsGetter`是对kubernetes中`pod`对象增删改查操作的接口。`ServicesGetter`是对`service`对象的操作的接口。

### ***\*PodsGetter\****

`PodsGetter` 是 `CoreV1Interface` 中的一个接口，用于对 Kubernetes 中的 Pod 资源对象进行增删改查操作。该接口包含以下方法：

+ `Pods(namespace string) PodInterface`：返回一个 `PodInterface` 对象，用于对指定命名空间中的 Pod 资源对象进行增删改查操作。

`PodInterface` 对象包含了多个方法，用于对 Pod 资源对象进行增删改查操作，例如：

+ `Create(*v1.Pod) (*v1.Pod, error)`：创建一个新的 Pod 资源对象。
+ `Update(*v1.Pod) (*v1.Pod, error)`：更新一个已有的 Pod 资源对象。
+ `Delete(name string, options *metav1.DeleteOptions) error`：删除一个指定名称的 Pod 资源对象。
+ `Get(name string, options metav1.GetOptions) (*v1.Pod, error)`：获取一个指定名称的 Pod 资源对象。
+ `List(opts metav1.ListOptions) (*v1.PodList, error)`：获取一个指定命名空间中的所有 Pod 资源对象列表。

💡简单的一个案例如下：

```go
pods, err := clientset.CoreV1().Pods("").List(metav1.ListOptions{})
```

**CoreV1().Pods()**

+ [k8s.io/client-go/kubernetes/typed/core/v1/core_client.go](https://github.com/kubernetes/client-go/blob/87887458218a51f3944b2f4c553eb38173458e97/kubernetes/typed/core/v1/core_client.go#L87)

```go
func (c *CoreV1Client) Pods(namespace string) PodInterface {
    return newPods(c, namespace)
}
```

**newPods()**

```go
// newPods returns a Pods
func newPods(c *CoreV1Client, namespace string) *pods {
    return &pods{
        client: c.RESTClient(),
        ns:     namespace,
    }
}
```

> `CoreV1().Pods()`的方法实际上是调用了`newPods()`的方法，创建了一个`pods`对象，`pods`对象继承了`rest.Interface`接口，即最终的实现本质是`RESTClient`的HTTP调用。

+ [k8s.io/client-go/kubernetes/typed/core/v1/pod.go](https://github.com/kubernetes/client-go/blob/87887458218a51f3944b2f4c553eb38173458e97/kubernetes/typed/core/v1/pod.go#L48)

```go
// pods implements PodInterface
type pods struct {
    client rest.Interface
    ns     string
}
```

`pods`对象实现了`PodInterface`接口。`PodInterface`定义了`pods`对象的增删改查等方法。

```go
// PodInterface has methods to work with Pod resources.
type PodInterface interface {
    Create(*v1.Pod) (*v1.Pod, error)
    Update(*v1.Pod) (*v1.Pod, error)
    UpdateStatus(*v1.Pod) (*v1.Pod, error)
    Delete(name string, options *meta_v1.DeleteOptions) error
    DeleteCollection(options *meta_v1.DeleteOptions, listOptions meta_v1.ListOptions) error
    Get(name string, options meta_v1.GetOptions) (*v1.Pod, error)
    List(opts meta_v1.ListOptions) (*v1.PodList, error)
    Watch(opts meta_v1.ListOptions) (watch.Interface, error)
    Patch(name string, pt types.PatchType, data []byte, subresources ...string) (result *v1.Pod, err error)
    PodExpansion
}
```

**PodsGetter**

PodsGetter继承了PodInterface的接口。

```go
// PodsGetter has a method to return a PodInterface.
// A group's client should implement this interface.
type PodsGetter interface {
    Pods(namespace string) PodInterface
}
```

**Pods().List()**

pods.List()方法通过`RESTClient`的HTTP调用来实现对kubernetes的pod资源的获取。

```go
// List takes label and field selectors, and returns the list of Pods that match those selectors.
func (c *pods) List(opts meta_v1.ListOptions) (result *v1.PodList, err error) {
    result = &v1.PodList{}
    err = c.client.Get().
        Namespace(c.ns).
        Resource("pods").
        VersionedParams(&opts, scheme.ParameterCodec).
        Do().
        Into(result)
    return
}
```

### ***\*RESTClient\****

`RESTClient` 是客户端 API 的核心对象。它允许你使用 RESTful HTTP 操作来调用 kubernetes API，例如 GET、POST、PUT 和 DELETE 操作。`RESTClient` 对象的创建过程如下：

+ 创建一个 `rest.Config` 对象，该对象包含了与 Kubernetes API 服务器通信所需的配置信息，例如 API 服务器的地址、用户身份验证信息等。
+ 调用 `rest.RESTClientFor(&config)` 方法创建 `RESTClient` 对象。

创建 `RESTClient` 对象后，你可以使用其提供的各种方法调用 Kubernetes API，例如 `Get`、`Post`、`Put` 和 `Delete` 等。

`RESTClient` 通常是客户端 API 中使用最多的对象。它提供了一种方便、高效的方式来与 Kubernetes API 进行交互，可以用来查询、创建、更新和删除 kubernetes 中的各种资源对象，例如 Pod、Service 等。

在 Kubernetes 中，不同的资源对象对应了不同的 API 版本，因此 `RESTClient` 的方法通常也会按照 API 的版本进行分类，例如 `corev1.RESTClient` 提供了访问 Core API 的方法，`appsv1.RESTClient` 则提供了访问 Apps API 的方法，以此类推。

总之，`RESTClient` 是客户端 API 的最核心的对象之一，它提供了一种方便、高效的方式来与 Kubernetes API 进行交互，是开发 kubernetes 应用程序的必备组件之一。

+ [k8s.io/client-go/kubernetes/typed/core/v1/core_client.go](https://github.com/kubernetes/client-go/blob/87887458218a51f3944b2f4c553eb38173458e97/kubernetes/typed/core/v1/core_client.go#L121)

```go
client, err := rest.RESTClientFor(&config)
```

***\**\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*RESTClientFor\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\**\***

```go
// RESTClientFor returns a RESTClient that satisfies the requested attributes on a client Config
// object. Note that a RESTClient may require fields that are optional when initializing a Client.
// A RESTClient created by this method is generic - it expects to operate on an API that follows
// the Kubernetes conventions, but may not be the Kubernetes API.
func RESTClientFor(config *Config) (*RESTClient, error) {
    ...
    qps := config.QPS
    ...
    burst := config.Burst
    ...
    baseURL, versionedAPIPath, err := defaultServerUrlFor(config)
    ...
    transport, err := TransportFor(config)
    ...
    var httpClient *http.Client
    if transport != http.DefaultTransport {
        httpClient = &http.Client{Transport: transport}
        if config.Timeout > 0 {
            httpClient.Timeout = config.Timeout
        }
    }

    return NewRESTClient(baseURL, versionedAPIPath, config.ContentConfig, qps, burst, config.RateLimiter, httpClient)
}
```

`RESTClientFor`函数调用了`NewRESTClient`的初始化函数。

### ***\*NewRESTClient\****

+ [k8s.io/client-go/rest/client.go](https://github.com/kubernetes/client-go/blob/87887458218a51f3944b2f4c553eb38173458e97/rest/client.go#L91)

```go
// NewRESTClient creates a new RESTClient. This client performs generic REST functions
// such as Get, Put, Post, and Delete on specified paths.  Codec controls encoding and
// decoding of responses from the server.
func NewRESTClient(baseURL *url.URL, versionedAPIPath string, config ContentConfig, maxQPS float32, maxBurst int, rateLimiter flowcontrol.RateLimiter, client *http.Client) (*RESTClient, error) {
    base := *baseURL
    ...
    serializers, err := createSerializers(config)
    ...
    return &RESTClient{
        base:             &base,
        versionedAPIPath: versionedAPIPath,
        contentConfig:    config,
        serializers:      *serializers,
        createBackoffMgr: readExpBackoffConfig,
        Throttle:         throttle,
        Client:           client,
    }, nil
}
```

### ***\*RESTClient结构体\****

以下介绍RESTClient的结构体定义，RESTClient结构体中包含了`http.Client`，即本质上RESTClient就是一个`http.Client`的封装实现。

+ [k8s.io/client-go/rest/client.go](https://github.com/kubernetes/client-go/blob/87887458218a51f3944b2f4c553eb38173458e97/rest/client.go#L54)

```go
// RESTClient imposes common Kubernetes API conventions on a set of resource paths.
// The baseURL is expected to point to an HTTP or HTTPS path that is the parent
// of one or more resources.  The server should return a decodable API resource
// object, or an api.Status object which contains information about the reason for
// any failure.
//
// Most consumers should use client.New() to get a Kubernetes API client.
type RESTClient struct {
    // base is the root URL for all invocations of the client
    base *url.URL
    // versionedAPIPath is a path segment connecting the base URL to the resource root
    versionedAPIPath string

    // contentConfig is the information used to communicate with the server.
    contentConfig ContentConfig

    // serializers contain all serializers for underlying content type.
    serializers Serializers

    // creates BackoffManager that is passed to requests.
    createBackoffMgr func() BackoffManager

    // TODO extract this into a wrapper interface via the RESTClient interface in kubectl.
    Throttle flowcontrol.RateLimiter

    // Set specific behavior of the client.  If not set http.DefaultClient will be used.
    Client *http.Client
}
```

### ***\*RESTClient.Interface\****

RESTClient实现了以下的接口方法：

+ [k8s.io/client-go/rest/client.go](https://github.com/kubernetes/client-go/blob/87887458218a51f3944b2f4c553eb38173458e97/rest/client.go#L42)

```go
// Interface captures the set of operations for generically interacting with Kubernetes REST apis.
type Interface interface {
    GetRateLimiter() flowcontrol.RateLimiter
    Verb(verb string) *Request
    Post() *Request
    Put() *Request
    Patch(pt types.PatchType) *Request
    Get() *Request
    Delete() *Request
    APIVersion() schema.GroupVersion
}
```

在调用HTTP方法（Post()，Put()，Get()，Delete() ）时，实际上调用了Verb(verb string)函数。

```go
// Verb begins a request with a verb (GET, POST, PUT, DELETE).
//
// Example usage of RESTClient's request building interface:
// c, err := NewRESTClient(...)
// if err != nil { ... }
// resp, err := c.Verb("GET").
//  Path("pods").
//  SelectorParam("labels", "area=staging").
//  Timeout(10*time.Second).
//  Do()
// if err != nil { ... }
// list, ok := resp.(*api.PodList)
//
func (c *RESTClient) Verb(verb string) *Request {
    backoff := c.createBackoffMgr()

    if c.Client == nil {
        return NewRequest(nil, verb, c.base, c.versionedAPIPath, c.contentConfig, c.serializers, backoff, c.Throttle)
    }
    return NewRequest(c.Client, verb, c.base, c.versionedAPIPath, c.contentConfig, c.serializers, backoff, c.Throttle)
}
```

`Verb`函数调用了`NewRequest`方法，最后调用`Do()`方法实现一个HTTP请求获取Result。

### 认证问题

API Server 是支持双向 TLS 认证的，

双向 TLS 认证是指在建立 TLS 连接时，不仅客户端会验证服务器的身份，服务器也会验证客户端的身份。它通过使用客户端证书来实现客户端身份验证。在进行双向 TLS 认证时，客户端和服务器先各自生成自己的证书和私钥，然后将证书和公钥分别发送给对方。当客户端向服务器发送请求时，服务器会要求客户端提供证书进行身份验证。客户端会将自己的证书发送给服务器，服务器会使用该证书上的公钥来验证客户端的身份。如果验证成功，服务器会使用自己的证书和私钥对通信进行加密，确保通信过程中的安全性。

### ***\*调用discovery和dynamic API\****

可以使用`discovery`和`dynamic` API 来访问那些不在 Kubernetes API 中的自定义资源。`discovery` API 提供了一种方法来发现集群中的资源，而`dynamic` API 则提供了一种通用的方法来访问这些资源。使用这些 API，可以轻松地与 Kubernetes API 交互并访问自定义资源。Client-go 提供了广泛的功能，用于与 Kubernetes API 交互，包括强类型 API、资源客户端、Watch API 和动态客户端。

### 结论

Kubernetes API是与Kubernetes交互的REST API，而client-go是用于在Go中与这些API交互的库。使用client-go，开发人员可以轻松地在Kubernetes中创建、读取、更新和删除资源对象。

## 总结

Kubernetes是一个容器编排平台，简化了容器化应用程序的部署、扩展和管理。它提供了一种以声明方式管理容器化应用程序的方法，这意味着开发人员可以指定应用程序的期望状态，Kubernetes确保实现期望状态。Kubernetes提供了丰富的API与平台交互，client-go是与这些API交互最流行的库之一。

Client-go提供了广泛的功能，用于与Kubernetes API交互。一些关键功能如下：

Client-go提供了一个强类型API，用于与Kubernetes资源交互。这意味着开发人员可以使用Go类型系统与Kubernetes资源交互，这提供了更好的类型安全性并减少了运行时错误的可能性。

Client-go提供了资源客户端，用于与Kubernetes资源交互。这些客户端提供了一个简单且一致的接口，用于在Kubernetes资源上执行CRUD（创建、读取、更新、删除）操作。

Client-go提供了一个Watch API，允许开发人员监视Kubernetes资源的变化。这对于构建控制器和其他需要对Kubernetes环境中的变化做出反应的应用程序非常有用。

Client-go提供了一个动态客户端，允许开发人员与Kubernetes资源交互，而无需为每个资源生成代码。这对于构建需要与任意Kubernetes资源交互的通用工具和实用程序非常有用。

Client-go是Kubernetes生态系统的关键组成部分。它提供了一种以简单和一致的方式与Kubernetes API交互的方式。Client-go被许多流行的Kubernetes工具和框架使用，包括Kubernetes本身。如果没有client-go，构建和维护Kubernetes应用程序将更加困难。

Client-go是与Kubernetes API交互的强大而灵活的库。它提供了广泛的功能，用于与Kubernetes资源交互，包括强类型API、资源客户端、Watch API和动态客户端。Client-go是Kubernetes生态系统的关键组成部分，许多流行的Kubernetes工具和框架都在使用它。如果您正在使用Go构建Kubernetes应用程序，则client-go是必不可少的库。

### 调用过程

`client-go`对kubernetes资源对象的调用，需要先获取kubernetes的配置信息，即`$HOME/.kube/config`。

整个调用的过程如下：

```go
kubeconfig→rest.config→clientset→具体的client(CoreV1Client)→具体的资源对象(pod)→RESTClient→http.Client→HTTP请求的发送及响应
```

通过clientset中不同的client和client中不同资源对象的方法实现对kubernetes中资源对象的增删改查等操作，常用的client有`CoreV1Client`、`AppsV1beta1Client`、`ExtensionsV1beta1Client`等。

### **client-go对k8s资源的调用**

**创建clientset**

```
//获取kubeconfig
kubeconfig = flag.String("kubeconfig", filepath.Join(home, ".kube", "config"), "(optional) absolute path to the kubeconfig file")
//创建config
config, err := clientcmd.BuildConfigFromFlags("", *kubeconfig)
//创建clientset
clientset, err := kubernetes.NewForConfig(config)
//具体的资源调用见以下例子
```

## **deployment**

```go
//声明deployment对象
var deployment *v1beta1.Deployment
//构造deployment对象
//创建deployment
deployment, err := clientset.AppsV1beta1().Deployments(<namespace>).Create(<deployment>)
//更新deployment
deployment, err := clientset.AppsV1beta1().Deployments(<namespace>).Update(<deployment>)
//删除deployment
err := clientset.AppsV1beta1().Deployments(<namespace>).Delete(<deployment.Name>, &meta_v1.DeleteOptions{})
//查询deployment
deployment, err := clientset.AppsV1beta1().Deployments(<namespace>).Get(<deployment.Name>, meta_v1.GetOptions{})
//列出deployment
deploymentList, err := clientset.AppsV1beta1().Deployments(<namespace>).List(&meta_v1.ListOptions{})
//watch deployment
watchInterface, err := clientset.AppsV1beta1().Deployments(<namespace>).Watch(&meta_v1.ListOptions{})
```

## **service**

```
//声明service对象
var service *v1.Service
//构造service对象
//创建service
service, err := clientset.CoreV1().Services(<namespace>).Create(<service>)
//更新service
service, err := clientset.CoreV1().Services(<namespace>).Update(<service>)
//删除service
err := clientset.CoreV1().Services(<namespace>).Delete(<service.Name>, &meta_v1.DeleteOptions{})
//查询service
service, err := clientset.CoreV1().Services(<namespace>).Get(<service.Name>, meta_v1.GetOptions{})
//列出service
serviceList, err := clientset.CoreV1().Services(<namespace>).List(&meta_v1.ListOptions{})
//watch service
watchInterface, err := clientset.CoreV1().Services(<namespace>).Watch(&meta_v1.ListOptions{})
```

## **ingress**

```
//声明ingress对象
var ingress *v1beta1.Ingress
//构造ingress对象
//创建ingress
ingress, err := clientset.ExtensionsV1beta1().Ingresses(<namespace>).Create(<ingress>)
//更新ingress
ingress, err := clientset.ExtensionsV1beta1().Ingresses(<namespace>).Update(<ingress>)
//删除ingress
err := clientset.ExtensionsV1beta1().Ingresses(<namespace>).Delete(<ingress.Name>, &meta_v1.DeleteOptions{})
//查询ingress
ingress, err := clientset.ExtensionsV1beta1().Ingresses(<namespace>).Get(<ingress.Name>, meta_v1.GetOptions{})
//列出ingress
ingressList, err := clientset.ExtensionsV1beta1().Ingresses(<namespace>).List(&meta_v1.ListOptions{})
//watch ingress
watchInterface, err := clientset.ExtensionsV1beta1().Ingresses(<namespace>).Watch(&meta_v1.ListOptions{})
```

## **replicaSet**

```
//声明replicaSet对象
var replicaSet *v1beta1.ReplicaSet
//构造replicaSet对象
//创建replicaSet
replicaSet, err := clientset.ExtensionsV1beta1().ReplicaSets(<namespace>).Create(<replicaSet>)
//更新replicaSet
replicaSet, err := clientset.ExtensionsV1beta1().ReplicaSets(<namespace>).Update(<replicaSet>)
//删除replicaSet
err := clientset.ExtensionsV1beta1().ReplicaSets(<namespace>).Delete(<replicaSet.Name>, &meta_v1.DeleteOptions{})
//查询replicaSet
replicaSet, err := clientset.ExtensionsV1beta1().ReplicaSets(<namespace>).Get(<replicaSet.Name>, meta_v1.GetOptions{})
//列出replicaSet
replicaSetList, err := clientset.ExtensionsV1beta1().ReplicaSets(<namespace>).List(&meta_v1.ListOptions{})
//watch replicaSet
watchInterface, err := clientset.ExtensionsV1beta1().ReplicaSets(<namespace>).Watch(&meta_v1.ListOptions{})
```

新版的kubernetes中一般通过deployment来创建replicaSet，再通过replicaSet来控制pod。

## **pod**

```go
//声明pod对象
var pod *v1.Pod
//创建pod
pod, err := clientset.CoreV1().Pods(<namespace>).Create(<pod>)
//更新pod
pod, err := clientset.CoreV1().Pods(<namespace>).Update(<pod>)
//删除pod
err := clientset.CoreV1().Pods(<namespace>).Delete(<pod.Name>, &meta_v1.DeleteOptions{})
//查询pod
pod, err := clientset.CoreV1().Pods(<namespace>).Get(<pod.Name>, meta_v1.GetOptions{})
//列出pod
podList, err := clientset.CoreV1().Pods(<namespace>).List(&meta_v1.ListOptions{})
//watch pod
watchInterface, err := clientset.CoreV1().Pods(<namespace>).Watch(&meta_v1.ListOptions{})
```

## **statefulset**

```go
//声明statefulset对象
var statefulset *v1.StatefulSet
//创建statefulset
statefulset, err := clientset.AppsV1().StatefulSets(<namespace>).Create(<statefulset>)
//更新statefulset
statefulset, err := clientset.AppsV1().StatefulSets(<namespace>).Update(<statefulset>)
//删除statefulset
err := clientset.AppsV1().StatefulSets(<namespace>).Delete(<statefulset.Name>, &meta_v1.DeleteOptions{})
//查询statefulset
statefulset, err := clientset.AppsV1().StatefulSets(<namespace>).Get(<statefulset.Name>, meta_v1.GetOptions{})
//列出statefulset
statefulsetList, err := clientset.AppsV1().StatefulSets(<namespace>).List(&meta_v1.ListOptions{})
//watch statefulset
watchInterface, err := clientset.AppsV1().StatefulSets(<namespace>).Watch(&meta_v1.ListOptions{})
```

### secret

```go
//声明secret对象
var secret *v1.Secret

//创建secret
secret, err := clientset.CoreV1().Secrets(<namespace>).Create(<secret>)
//更新secret
secret, err := clientset.CoreV1().Secrets(<namespace>).Update(<secret>)
//删除secret
err := clientset.CoreV1().Secrets(<namespace>).Delete(<secret.Name>, &meta_v1.DeleteOptions{})
//查询secret
secret, err := clientset.CoreV1().Secrets(<namespace>).Get(<secret.Name>, meta_v1.GetOptions{})
//列出secret
secretList, err := clientset.CoreV1().Secrets(<namespace>).List(&meta_v1.ListOptions{})
//watch secret
watchInterface, err := clientset.CoreV1().Secrets(<namespace>).Watch(&meta_v1.ListOptions{})
```

### configmap

```go
//声明configmap对象
var configMap *v1.ConfigMap

//创建configmap
configMap, err := clientset.CoreV1().ConfigMaps(<namespace>).Create(<configMap>)
//更新configmap
configMap, err := clientset.CoreV1().ConfigMaps(<namespace>).Update(<configMap>)
//删除configmap
err := clientset.CoreV1().ConfigMaps(<namespace>).Delete(<configMap.Name>, &meta_v1.DeleteOptions{})
//查询configmap
configMap, err := clientset.CoreV1().ConfigMaps(<namespace>).Get(<configMap.Name>, meta_v1.GetOptions{})
//列出configmap
configMapList, err := clientset.CoreV1().ConfigMaps(<namespace>).List(&meta_v1.ListOptions{})
//watch configmap
watchInterface, err := clientset.CoreV1().ConfigMaps(<namespace>).Watch(&meta_v1.ListOptions{})
```

通过以上对kubernetes的资源对象的操作函数可以看出，每个资源对象都有增删改查等方法，基本调用逻辑类似。一般二次开发只需要创建deployment、service、ingress三个资源对象即可，pod对象由deployment包含的replicaSet来控制创建和删除。函数调用的入参一般只有`NAMESPACE`和`kubernetesObject`两个参数，部分操作有`Options`的参数。在创建前，需要对资源对象构造数据，可以理解为编辑一个资源对象的yaml文件，然后通过`kubectl create -f xxx.yaml`来创建对象。
