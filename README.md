# Rancher Catalogs应用商店模版编写介绍

## Rancher Catalogs应用商店介绍
在Rancher中提供基于helm的应用商店，通过应用商店能够快速和容易的重复部署应用。
Rancher自身改进了Helm目录和charts。所有原生的Helm charts都可以在Rancher中运行，但为了提升用户体验，Rancher增加了一些功能。在Rancher中提供了由Rancher官方维护的模板仓库、由Kubernetes社区维护的模板仓库和未经测试验证的应用模板仓库这三种内置的应用商店。另外，也支持添加自定义的应用商店。
Rancher支持两种charts类型：
- Helm Charts：原生的Helm charts包含运行应用的所有内容。当部署一个原生的Helm charts时，需要通过Answers配置键值对形式的参数。
- Rancher charts在原生的helm charts基础上，添加量app-readme.md和questions.yaml这两个文件，以提升用户体验。 
## Rancher Chart 的基本结构
Helm的打包格式叫做chart，所谓chart就是一系列文件, 它描述了一组相关的 k8s 集群资源。Rancher Chart中的文件安装特定的目录结构组织, 最简单的chart 目录如下所示：
```yaml
wordpress
├── charts                       #charts目录存放依赖的chart
└── answers.yaml                 #配置键值对形式的参数
    ├── Chart.yaml               #包含chart信息的YAML文件
    ├── README.md 				 #可选：chart的介绍信息等
    ├── app-readme.md		     #可选：Rancher应用商店页面中app描述信息。
    ├── requirements.yaml		 #可选：列举当前chart的需要依赖的Chart
    ├── questions.yaml		     #可选：Rancher应用商店图形化配置交互
    ├── templates				 #存放chart所有的K8s资源定义模板
    │   ├── deployment.yaml		 #kubernetes deployment模版
    │   ├── ingress.yaml		 #kubernetes ingress模版
    │   ├── secrets.yaml         #kubernetes secrets模版
    │   ├── configmap.yaml       #kubernetes configmap模版
    │   ├── pvc.yaml             #kubernetes PersistentVolumeClaims模版
    │   ├── NOTES.txt			 #部署chart后输出的帮助文档
    │   └── service.yaml		 #kubernetes service模版
    └── values.yaml				 #当前 Chart 的默认配置的值
```

## 编写一个简单的Helm Chart示例
### Chart.yaml文件编写

1、Chart.yaml 文件是 一个 chart 必要文件， 该文件可以简单包括以下字段;

```yaml
apiVersion: v1	                       #chart的API版本, 总是"v1",[必选]
name: wordpress		                   #chart的名称,[必选]
version: 2.1.12  	                   #chart的版本，这个版本必须必要遵循SemVer 2标准,[必选]
kubeVersion: 1.15.3                    #kubernetes的版本,遵循SemVer标准,[可选]
description: helm chart for mysql      #chart模板的简介描述,[可选]
keywords:                              #关于此项目关键字列表,[可选]
  - wordpress
  - cms
  - blog
  - http
  - web
  - application
  - php
home: https://github.com/WalkCloud	          #此项目的URL主页地址,[可选]
sources: https://github.com/WalkCloud/sources #此项目源代码的URL地址,[可选]
icon: https://github.com/WalkCloud/xxx.png	  #此项目图标路径：SVG或者png格式,[可选]
maintainers:                                  #维护人员信息,[可选]
  - email: hongyu@rancher.com                 #维护人员邮箱,[可选]      
    name: kevin lee                           #维护人员姓名,[可选]
    url: http://kevin.WalkCloud.com           #维护人员网址,[可选]
engine: gotpl                                 #模板引擎,[可选, 默认gotpl]
appVersion: 4.9.8                             #此包含的应用程序的版本
tillerVersion: 2.14.3                         #此版本需要请求的tiller的版本,[可选]
```
### valus.yaml文件编写
此文件用于传递配置信息到templates里的kubernetes yaml部署文件，编写建议如下：
- 使用2个空格作缩进；
- 确认数字为字符类型时，使用双引号引起来；
- 为了迎合helm3的规范，空定义最好将相关符号补上：
```
string: ""
list: []
map: {}
```
如果没什么特殊要求，一般需要修改的地方有image、service、healthCheck、persistentVolume.mountPaths等就基本满足应用发布需求。
```yaml
replicaCount: 1                    #用于配置容器副本数量，
statefulset:                       #配置statefulset有状态的应用负载 
  enabled: false                   #statefulset关闭
image:                             #用于配置kubernetes容器镜像，
  registry: private.harbor.com     #配置私有的image仓库地址，默认docker.io
  repository: bitnami/wordpress    #image名称，例如：nginx镜像
  tag: 4.9.8-debian-9              #image tag
  pullPolicy: IfNotPresent         #pull镜像的方式
service:                           #用于修改kubernetes Service
  type: ClusterIP                  #Service访问类型.
  port: 80                         #应用端口暴露
ingress:                           #用于修改kubernetes ingress  
  enabled: false                   #ingress功能开启
  annotations: {}                  #ingress的宣告信息
  path: /
  hosts:
    - chart-example.local
  tls: []
env: []
#  - name: DEMO_GREETING
#    value: "Hello from the environment"
#  - name: DEMO_FAREWELL
#    value: "Such a sweet sorrow"
config: []
#  enabled: true
#  mountPath: /conf 
#  data:
#    app.conf: |-
#      appname = example-chart
#    bpp.conf: |-
#      bppname
persistentVolume:                  #用于配置kubernetes持久化存储
  enabled: true                    #pv功能开启
# storageClass: "-"                #storageclass设置
  accessMode: ReadWriteOnce        #pv权限编辑
  size: 8Gi                        #pv容量设置
  mountPaths: []
  #  - name: data-storage
  #    mountPath: /config
  #    subPath: config
  #  - name: data-storage
  #    mountPath: /data
  #    subPath: data
resources:                         #用于kubenretes资源限制和配额
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 100m
    memory: 128Mi
healthCheck:                        #用于配置kubernetes健康检查
  enabled: true                     #健康检查功能开启
  type: tcp                         #http/tcp
  port: http                        #健康检查的端口名或端口
  httpPath: '/'                     #http时必须设置
  livenessInitialDelaySeconds: 10   #初始延迟秒数
  livenessPeriodSeconds: 10         #检测周期，默认值10，最小为1
  readinessInitialDelaySeconds: 10  #初始延迟秒数
  readinessPeriodSeconds: 10        #检测周期，默认值10，最小为1
nodeSelector: {}
tolerations: []
affinity: {}
```
### requirements.yaml文件编写
管理charts依赖，比如 WordPress Chart 依赖于 mariadb Chart
```yaml    
ependencies:                                                #配置依赖          
- name: mariadb                                             #依赖的charts名称[必选]
  version: 5.x.x                                            #依赖的charts版本[必选]
  repository: https://github.com/WalkCloud/mariadb/charts   #依赖的charts存储库URL路径
  condition: mariadb.enabled
  tags:
    - wordpress-database
```
### questions.yaml文件编写

questions.yam文件编写配置参考如下：

| 变量                | 类型          | 请求  | 描述                                                         |
| :------------------ | :------------ | :---- | :----------------------------------------------------------- |
| variable            | string        | true  | 定义“ values.yml”文件中指定的变量名，对嵌套对象使用“ foo.bar”。 |
| label               | string        | true  | 定义UI标签                                                   |
| description         | string        | false | 指定变量名的描述                                             |
| type                | string        | false | 如果未指定，则默认为`string`（当前支持的类型为string，boolean，int，enum，password，storageclass和hostname）。 |
| required            | bool          | false | 定义变量名是否被请求或者不被请求。(true \| false)            |
| default             | string        | false | 指定默认值                                                   |
| group               | string        | false | Group questions by input value.                              |
| min_length          | int           | false | 最小字节长度                                                 |
| max_length          | int           | false | 最大字节长度                                                 |
| min                 | int           | false | 最小整数长度                                                 |
| max                 | int           | false | 最大整数长度                                                 |
| options             | []string      | false | 在变量类型为“enum”时指定选项，例如：选项：-“ ClusterIP”-“ NodePort”-“ LoadBalancer” |
| valid_chars         | string        | false | 用于输入字符验证的正则表达式。                               |
| invalid_chars       | string        | false | 用于输入无效字符验证的正则表达式                             |
| subquestions        | []subquestion | false | Add an array of subquestions.                                |
| show_if             | string        | false | 如果条件变量为true，则显示当前变量。例如： `show_if: "serviceType=Nodeport"` |
| show_subquestion_if | string        | false | 显示子问题是否为真或等于选项之一。例如： `show_subquestion_if: "true"` |

questions.yaml示例如下：

```yaml
categories:
- Blog
- CMS
questions:
- variable: defaultImage
  default: true
  description: "Use default Docker image"
  label: Use Default Image
  type: boolean
  show_subquestion_if: false
  group: "Container Images"
  subquestions:
  - variable: image.repository
    default: "bitnami/wordpress"
    description: "WordPress image name"
    type: string
    label: WordPress Image Name
  - variable: image.tag
    default: "4.9.8-debian-9"
    description: "WordPress image tag"
    type: string
    label: Image Tag
  - variable: mariadb.image.repository
    default: "bitnami/mariadb"
    description: "MariaDB image name"
    type: string
    label: MariaDB Image Name
  - variable: mariadb.image.tag
    default: "10.1.35-debian-9"
    description: "MariaDB image tag"
    type: string
    label: MariaDB Image Tag

```
### answers.yaml文件编写
当部署一个原生的Helm charts时，需要通过Answers配置键值对形式的参数
```yaml
defaultImage: true
image.repository: bitnami/wordpress
image.tag: 4.9.8-debian-9
mariadb.image.repository: bitnami/mariadb
mariadb.image.tag: 10.1.35-debian-9
```
### templates文件夹内kubernetes yaml文件编写

此文件夹为kubernetes各种控制器配置的模版文件夹，所有控制器需要配置的yaml文件主要参数由valus.yaml文件传入。
templates文件夹里面的yaml文件使用 {{ }} 符号的是 Go 模板语言的标准。其中可以通过：
- .Values 对象访问 values.yaml 文件的内容， 前面的dot(.) 表示从顶层命名空间开始，找到 Values 对象(下同)
- .Release、.Chart 开头的预定义值可用于任何的模板中
- .Chart 对象用来访问 Chart.yaml 文件的内容
- .Release 对象是 Helm的内置对象之一， 使用 Helm 安装一个 release 时，由 Tiller 分配 release 的名称

由于templates文件夹里的文件众多，此例中通过deployments.yaml和service.yaml文件用于示例其他yaml表示方式
```yaml
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: {{ template "fullname" . }}
  labels:
    app: {{ template "fullname" . }}
    chart: "{{ .Chart.Name }}-{{ .Chart.Version }}"
    release: {{ .Release.Name }}
    heritage: {{ .Release.Service }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ template "fullname" . }}
      release: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ template "fullname" . }}
        release: {{ .Release.Name }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /
              port: http
          readinessProbe:
            httpGet:
              path: /
              port: http
          resources:
{{ toYaml .Values.resources | indent 12 }}
    {{- with .Values.nodeSelector }}
      nodeSelector:
{{ toYaml . | indent 8 }}
    {{- end }}
    {{- with .Values.affinity }}
      affinity:
{{ toYaml . | indent 8 }}
    {{- end }}
    {{- with .Values.tolerations }}
      tolerations:
{{ toYaml . | indent 8 }}
    {{- end }}
```
```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ template "fullname" . }}
  labels:
    app: {{ template "fullname" . }}
    chart: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
    release: {{ .Release.Name }}
    heritage: {{ .Release.Service }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http 
      protocol: TCP
      name: http
  selector:
    app: {{ template "fullname" . }}
    release: {{ .Release.Name }}
```
