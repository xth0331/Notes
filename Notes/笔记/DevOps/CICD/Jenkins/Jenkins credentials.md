# Jenkins credentials

许多第三方站点和应用可以与Jenkins交互，应用的管理员可以在应用中配置凭据以供Jenkins使用，通常是通过这些凭据应用访问控制。Jenkins管理员在Jenkins中添加/配置这些凭据，管道项目就可以使用凭据与这些第三方应用程序进行交互。

> 注意：Jenkins凭据功能由Credentials Binding插件提供。

可以使用存储在Jenkins中的凭据：

- 任何适用于整个Jenkins的地方（全局证书），
- 通过特定的Pipeline项目，
- 由特定的Jenkins用户。

Jenkins可以存储以下类型的凭据：

- **Secret text** - 诸如API令牌之类的令牌（例如GitHub个人访问令牌），
- **用户名和密码** - 可以作为单独的组件处理，也可以作为格式的冒号分隔字符串[处理](https://jenkins.io/doc/book/pipeline/jenkinsfile#handling-credentials)`username:password`（在[处理凭据中](https://jenkins.io/doc/book/pipeline/jenkinsfile#handling-credentials)详细了解此信息 ），
- **Secret file**  - 本质上是文件中的秘密内容，
- **带有私钥的SSH用户名** - [SSH公钥/私钥对](http://www.snailbook.com/protocols.html)，
- **证书** - [PKCS＃12证书文件](https://tools.ietf.org/html/rfc7292)和可选密码，或
- **Docker主机证书身份验证**凭据。

## 凭据安全

为了最大限度地提高安全性，Jenkins中配置的凭据以加密形式存储在主Jenkins实例上，并且仅通过其凭据ID在Pipeline项目中处理。

## 配置凭据

如果是默认设置，将获得对Jenkins的全部控制权限，如果授权策略为其他，则需要确保Jenkins拥有凭据的创建权限。



![Jenkins-Authorization](https://blog-image.nos-eastchina1.126.net/Jenkins-Authorization.jpg)



## 添加新的全局凭据

要向Jenkins实例添加新的全局凭据：

1. 如果需要，请确保您已登录Jenkins（作为具有 **凭据>创建**权限的用户）。
2. 在Jenkins主页（即Jenkins经典UI的仪表板）中，单击左侧的**凭据>系统**。
3. 在“ **系统”下**，单击“ **全局凭据（不受限制）”**链接以访问此默认域。
4. 单击左侧的“ **添加凭据** ”。
   **注意：**如果此默认域中没有凭据，您还可以单击“ **添加一些凭据”**链接（与单击“ **添加凭据”**链接相同）。
5. 在**Kind**字段中，选择 要添加[的凭据类型](https://jenkins.io/doc/book/using/using-credentials/#types-of-credentials)。
6. 在**Scope**字段中，选择以下任一项：
   - **全局** - 如果要添加的凭证是/是否为管道项目/项目。选择此选项会将crendentia/s的范围应用于Pipeline项目/项“对象”及其所有后代对象。
   - **系统** - 如果要添加的凭据是/ Jenkins实例本身与系统管理功能交互，例如电子邮件身份验证，代理连接等。选择此选项将crendential/s的范围应用于单个对象只要。
7. 将凭据本身添加到所选凭据类型的相应字段中：
   - **秘密文本** - 复制秘密文本并将其粘贴到“ **密码”**字段中。
   - **用户名和密码** - 在各自的字段中指定凭证的**用户名**和**密码**。
   - **秘密文件** -单击**选择文件**按钮旁边的**文件**字段选择秘密文件上传到詹金斯。
   - **具有私钥的SSH用户名** - 将凭据**用户名**， **私钥**和可选**密码短语指定**到其各自的字段中。
     **注意：****直接**选择**Enter**可以复制私钥的文本并将其粘贴到生成的**密钥**文本框中。
   - **证书** - 指定**证书**和可选**密码**。选择上 **载PKCS＃12证书**允许您通过生成的上**载证书**按钮将证书上载为文件。
   - **Docker主机证书身份验证** - 将相应的详细信息复制并粘贴到**客户端密钥**，**客户端证书**和**服务器CA证书**字段中。
8. 在**ID**字段中，指定有意义的凭据ID值 - 例如， `jenkins-user-for-xyz-artifact-repository`。您可以使用大写或小写字母作为凭据ID，以及任何有效的分隔符。但是，为了Jenkins实例上的所有用户的利益，最好使用单一且一致的约定来指定凭据ID。
   **注意：**此字段是可选的。如果未指定其值，Jenkins将为凭据ID分配全局唯一ID（GUID）值。请记住，一旦设置了凭证ID，就无法再更改凭证ID。
9. 为凭证指定可选**描述**。
10. 单击“ **确定”**以保存凭据。