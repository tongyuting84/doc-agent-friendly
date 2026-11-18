本文为您介绍使用容器服务（VKE）前，您需要完成的一系列准备工作。

## 步骤一：注册火山引擎

> **说明**

> 若您已注册火山引擎，请跳过此步骤。

1. 访问 [火山引擎控制台](https://console.volcengine.com/auth/login/)，单击 **现在就注册**。

2. 根据提示设置和填写账号信息，完成注册。更多信息，请参见 [如何进行账号注册](https://www.volcengine.com/docs/6261/64925)。

   注册成功后，单击 **前往登录页**，输入您上一步设置的账号密码，登录火山引擎控制台。

3. 完成账号实名认证。详细操作，请参见 [实名认证](https://www.volcengine.com/docs/6261/64935)。

## 步骤二：充值账户

请确保您的火山引擎账户余额（含代金券）不低于 100 元，且除去代金券后账户余额不为零。详细操作，请参见 [费用中心。](https://www.volcengine.com/docs/6269/100429)

> **注意**

> 本快速入门操作中，创建 VKE 集群时可能会产生负载均衡、NAT 网关、公网 IP、云盘等云基础资源费用。请在创建集群页面下方的 **配置费用** 处，查看实际计费。更多计费说明及云基础资源计费，请参见 [产品计费](https://www.volcengine.com/docs/6460/70612)。

## 步骤三：开通容器服务权限

1. 登录 [容器服务控制台](https://console.volcengine.com/vke)。

2. 根据界面提示，申请容器服务控制台权限。

## 步骤四：（可选）创建和授权 IAM 用户

默认情况下，火山引擎主账号拥有容器服务产品的所有权限。若需要更多用户协同操作管理您帐号下的资源，您需要创建 IAM（Identity and Access Management，访问控制）用户，并为 IAM 用户（User）授予相应的角色和权限。

1. 登录 [访问控制控制台](https://console.volcengine.com/iam/overview/)。

2. 参考 [创建用户并授权](https://www.volcengine.com/docs/6257/94013) 内容，创建 IAM 用户，并为用户授权如下策略。

   更多策略相关介绍和配置，请参见 [授权管理](https://www.volcengine.com/docs/6460/101024)。

   |策略 |策略描述 |
   |---|---|
   |VKEFullAccess |该策略为容器服务全读写访问权限，包括云服务器（ECS）、负载均衡（CLB）、私有网络（VPC）、NAT 网关的全读写访问权限。 |
   |VKEInnerFullAccess |该策略为容器服务内部全读写访问权限。 |
   |VKEReadOnlyAccess |该策略为容器服务只读访问权限。 |
   |VKEInnerReadOnlyAccess |该策略为容器服务内部只读访问权限。 |
   |BillingCenterPayOrderAccess |该策略为费用中心收银台支付权限、OpenAPI 的 PayOrder 权限。 |

   > **注意**

   > 自 2025 年 08 月 31 日起，所有子账号（即使拥有 **FullAccess** 权限）在申请预付费资源时必须具备 **BillingCenterPayOrderAccess** 权限，否则将无法完成订单支付。策略相关更多说明，请参见 [权限管理](https://www.volcengine.com/docs/6269/1186807)。

## 步骤五：（可选）配置 RBAC 权限

容器服务提供 RBAC（Role\-Based Access Control）授权功能，满足企业用户细粒度的资源访问权限控制需求。授权 IAM 用户后，可以为 IAM 用户配置 RBAC 权限。

> **说明**

> 容器服务也支持为 IAM 角色（ Role）授予 RBAC 权限，本快速入门以火山引擎主账号身份为上方步骤四中创建的 IAM 用户（User）授权 RBAC 权限为例。更多详细的说明，请参见 [配置 RBAC 权限](https://www.volcengine.com/docs/6460/101023)。

1. 登录 [容器服务控制台](https://console.volcengine.com/vke)。

2. 在左侧导航栏单击 **授权管理** \> **用户授权**，找到需要 RBAC 授权的 IAM 用户，然后单击右侧 **操作** 列下的 **添加授权**。

3. 在 **添加授权** 界面，为用户/角色添加相应的权限。设置完成后单击 **确定**，添加完成权限。

## 步骤六：创建私有网络

私有网络 VPC 为容器服务中的集群提供一个隔离的、用户自主配置和管理的私有网络环境。

1. 登录 [私有网络控制台](https://console.volcengine.com/vpc)。

2. 在顶部导航栏，选择靠近您业务的地域。

3. 在左侧导航栏单击 **私有网络**，然后单击 **创建私有网络**。

4. 根据界面提示，配置私有网络和子网参数。详细的参数说明，请参见 [创建私有网络](https://www.volcengine.com/docs/6401/69467)。

   > **说明**

   > 本快速入门以创建名为 **demo\-vpc** 的私有网络，并在该私有网络下创建两个 **demo\-vpc\-subnetA**、**demo\-vpc\-subnetB** 子网为例。

## 步骤七：上传镜像

容器服务支持使用火山引擎镜像仓库（CR）中的镜像，也支持使用第三方镜像仓库中的镜像。使用容器服务前，请确保已上传镜像至镜像仓库。

> **说明**

> 本快速入门以部署 Nginx 应用为例，您可以使用文中提供的示例镜像，也可以自行上传 Nginx 镜像到 CR。详细操作，请参见 [推送和拉取镜像](https://www.volcengine.com/docs/6420/68645)。
