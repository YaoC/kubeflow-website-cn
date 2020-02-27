+++
title = "身份认证及 TLS 支持"
description = "使用你的自定义域名添加 TLS 支持和身份认证"
weight = 90
+++

本章节展示如何添加 TLS 支持以及在亚马逊网站服务（AWS）上使用你的自定义域名创建一个用户池来认证用户。

## 流量流转

外部流量 → [ Ingress → Istio ingress 网关 → Istio 虚拟服务 ]

当你创建和更新 Kubernetes 资源，一个 ingress 也会被创建出来用于管理从外部到 Kubernetes 服务的流量。AWS 应用负载均衡器（ALB）的 Ingress 控制器会提供一个 ALB 给这个 ingress。
默认情况下，TLS 以及身份认证在创建时不会被启用。

在 Kubeflow 0.6 版本，社区已经从 [Ambassador](https://www.getambassador.io/) 迁移到 [Istio](https://istio.io/) 来管理内部流量。
AWS 的解决方案是，TLS 和身份认证可以在 ALB 处理，授权可以在 Istio 层处理。

## 启用 TLS 和身份认证

目前，ALB 公共 DNS 名称证书不再被支持。替换方案是，你必须准备一个自定义域名。你可以在 Route53 或任何域名提供商如 [GoDaddy.com](https://www.godaddy.com/) 那注册你的域名。

[AWS 证书管理器](https://aws.amazon.com/certificate-manager/) 是一个能让你轻松提供、管理和部署公共和私有 SSL/TLS 证书从而在 AWS 服务和你的内部连接资源中使用的服务。

为了从 ALB Ingress 控制器获取 TLS 支持，你需要跟着 [这个教程](https://docs.aws.amazon.com/acm/latest/userguide/gs-acm-request-public.html) 在 AWS 证书管理器中申请一个证书。成功通过校验后，你将会获得一个 ARN 证书用来在 ALB Ingress 控制器使用。

<img src="/docs/images/aws/cognito-certarn.png"
  alt="Cognito Certificate ARN"
  class="mt-3 mb-3 border border-info rounded">

[AWS Cognito](https://aws.amazon.com/cognito/) 让你能快速且便捷地给你的网站和移动应用添加用户 注册、登录和访问控制。亚马逊 Cognito 能扩展到百万级别的用户并且支持使用社交身份提供商登录，比如脸书、谷歌和亚马逊，以及通过 SAML 2.0 协议认证的企业身份提供商。

<img src="/docs/images/aws/cognito-appclient.png"
  alt="Cognito Application Client Setting"
  class="mt-3 mb-3 border border-info rounded">

<img src="/docs/images/aws/cognito-domain.png"
  alt="Cognito Domain Name"
  class="mt-3 mb-3 border border-info rounded">

为了在 Kubeflow 认证并管理用户，让我们来创建一个用户池。你可以跟着下面这些步骤来做。当一个用户池创建好后，我们会得到一个 `UserPoolId`、一个 Cognito 域名的名称和一个 Cognito Pool Arn。

在你 `generate all -V` 之前，请在你的 Kubeflow 配置文件 `${CONFIG_FILE}` 中更新 Cognito spec 部分，以便它看起来像这样：

```
plugins:
    - name: aws
      spec:
        auth:
          cognito:
            cognitoUserPoolArn: arn:aws:cognito-idp:us-west-2:xxxxx:userpool/us-west-2_xxxxxx
            cognitoAppClientId: xxxxxbxxxxxx
            cognitoUserPoolDomain: your-user-pool
            certArn: arn:aws:acm:us-west-2:xxxxx:certificate/xxxxxxxxxxxxx-xxxx
        roles:
          - eksctl-kubeflow-aws-nodegroup-ng-a2-NodeInstanceRole-xxxxx
        region: us-west-2
```

完成 TLS 和身份认证配置之后，你就可以运行 `kfctl apply -V -f ${CONFIG_FILE}` 了。

在你的 ingress DNS 准备好后，你需要在你的 DNS 记录中创建一个 `CNAME`。

<img src="/docs/images/aws/custom-domain-cname.png"
  alt="Custom Domain CNAME"
  class="mt-3 mb-3 border border-info rounded">

然后你就可以访问 `https://www.shanjiaxin.com` 了，这个是我们用来在这做样例的自定义域名，它会跳转到一个身份认证页面。我们在 Cognito 设置中添加了一个用户 `kubeflow-test-user`，我们可以用这个用户来登录。

<img src="/docs/images/aws/authentication.png"
  alt="Cognito Authentication pop-up"
  class="mt-3 mb-3 border border-info rounded">

<img src="/docs/images/aws/kubeflow-main-page.png"
  alt="Kubeflow main page"
  class="mt-3 mb-3 border border-info rounded">