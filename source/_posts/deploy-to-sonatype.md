---
title: 从0发布一个构建到 Maven 中央仓库
date: 2018-07-10 19:58:08
categories: Java
tags:
 - Maven
 - Sonatype
 - Java
permalink: deploy-to-sonatype
---

> 原创内容

在这之前一直使用 Nexus OSS 的搭建私有仓库，今天想起折腾一下如何将一个构建发布到 Maven 的中央仓库当中，于是有了下面的云云……

{% asset_img sonatype.jpg [Sonatype 官网] %}

---

# 从0发布一个构建到Maven中央仓库


## 注册 Sonatype 账号

* Sonatype官网：http://www.sonatype.org/
* 注册地址：https://issues.sonatype.org/secure/Signup!default.jspa
* oss地址：https://oss.sonatype.org

## 登录 Sonaytype Jira 创建一个 issue

**创建一个 issue**

1. Project 选择 Community Support - Open Source Project Repository Hosting (OSSRH)
1. Issue Type 选择 new Project
1. Summary 输入你的 项目介绍
1. GroupId 填写你的工程使用的 groupId，如 com.github.xxx，如果是其他，必须是自己的域名，必要时，管理员会要求你用填写域名的邮箱发送验证邮件来证明域名是你的；
1. 其他不用填写，直接点击 Create，创建一个 issue
1. 接下来就是等待回复了；

**issue审批通过之后你会收到回复，like this：**

{% asset_img sonatype-issue.png [sonatype-issue] %}

这样就算是审批通过了，接下来就可以开始进入下一步发布构建了；

<!-- more -->

## 更新本地 Maven 的 setting.xml 文件

```xml
<!-- lang: xml -->
<settings>
    ...
    <servers>
        <server>
            <id>oss</id>
            <username><![CDATA[sonatype username]]></username>
            <password><![CDATA[password]]></password>
        </server>
    </servers>
    ...
</settings>
```

## 生成 GPG 签名

在上一步等待 issue 审核的过程中，我们也不要闲着，开始折腾如何生成 GPG 签名，这是后续必须要经历的流程；

**这里我只演示 macOS 下生成和上次 GPG 的流程，其他 OS 的童鞋请自行 google**

```shell
$ brew install gpg
....
$ gpg --version
gpg (GnuPG) 2.2.8
libgcrypt 1.8.3

## 安装完成，开始生成
$ gpg --gen-key
# 接下来会要求输入 username 和 email，请对号入座，确认信息之后会弹出一个 shell 对话框，要求输入签名秘钥passphrase，输入一个自己记得住的秘钥，一定要记下来，以后每次 deploy 都会使用到。确认后完成签名的生成。

# 生成之后会打印一堆信息；最好保存下来，其中：
....
gpg: 密钥 B3A4160E0484CEF0 被标记为绝对信任
....
B3A 开头的那个为签名的公钥ID，下面会用到。

# 可以通过下面的命令查看是否生成成功
$ gpg --list-keys

# 上传 GPG 公钥到秘钥服务器
$ gpg --keyserver hkp://pool.sks-keyservers.net --send-keys 公钥ID

# 通过下面的命令验证是否上传成功
$ gpg --keyserver hkp://pool.sks-keyservers.net --recv-keys 公钥ID

# 到这里，你可以去看看 sonatype issue 是不是已经有 comment 了
```

## 修改你的 POM 文件

在你的 pom.xml 文件中添加如下内容：

```
<profiles>
    <profile>
        <id>sonar</id>
        <build>
            <plugins>
                <!-- GPG -->
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-gpg-plugin</artifactId>
                    <version>${maven-gpg-plugin.version}</version>
                    <executions>
                        <execution>
                            <id>sign-artifacts</id>
                            <phase>verify</phase>
                            <goals>
                                <goal>sign</goal>
                            </goals>
                        </execution>
                    </executions>
                    <configuration>
                        <skip>${gpg.skip}</skip>
                    </configuration>
                </plugin>
            </plugins>
        </build>
        <distributionManagement>
            <snapshotRepository>
                <id>oss</id>
                <url>https://oss.sonatype.org/content/repositories/snapshots/</url>
            </snapshotRepository>
            <repository>
                <id>oss</id>
                <url>https://oss.sonatype.org/service/local/staging/deploy/maven2/</url>
            </repository>
        </distributionManagement>
    </profile>
</profiles>
```

当然你还应该有一些默认的 plugin，例如：maven-compiler-plugin 和 maven-source-plugin 也是必须的，请自行添加；

昨晚上面的这些，一切就已经准备就绪了，如果遇到什么错误，可以查看文末的常见错误解决方案；或者 google，作为一个 coder，一定要学会 google，不要告诉我你不会！！！

## 发布构建到 Sonatype

我 pom.xml 里面定义的 profiles id 为“sonar”，所以我执行下面的 Command

```
$ mvn clean deploy -Psonar

## 过程中会要求输入前面生成 GPG 秘钥的 passphrase，输入即可
```

如果执行报错，你可以添加 -e 打印具体什么错误，辅助解决问题；

如果不出意外，整个过程将会一帆风顺，刷过几屏日志之后，本地 Maven 构架就已经上传到 Sonatype 的 OSS 暂存仓库上了，注意只是StagingRepositories，还没有到中央仓库；

这是就可以登录 https://oss.sonatype.org/ 查看了
点击 左侧菜单栏当中的 Staging Repositiries，在右上角的搜索框中输入自己的 groupId 进行模糊检索；找到自己刚才 deploy 的构建；如下图所示：

{% asset_img oss-staging.png [OSS Staging Repositiries] %}

勾选构建，点击 Close ，输入一段 Description 之后点击 Confirm 按钮；
之后需要等待一段时间，等 Close Staging 的状态生效之后，就可以再次登录oss找到自己的构建，选中之后，点击 Release 按钮；再次 Confirm 之后，恭喜你，你的构建已经正式发布到 Maven 中央仓库了。

接下来，到中央仓库：https://search.maven.org/ 搜索到你的构建了，如果没有搜索到，稍等片刻一定会有的，OSS 同步到中央仓库需要一点点时间；

如果你已经在中央仓库搜索到自己的构建了，记得回到 sonatype 的 jira issue 里面回复管理员你已经完成了构建，可以 Close this issue 了；

### 意外的惊喜

当你走过上面的全部流程，你一定会觉得偌大的Sonatype发布为啥这么繁琐复杂，那你一定错了，这只是第一次，接下来你已经拥有了发布相同groupId构建的权限；不需要再提 issue，直接 deploy 你的 maven 构建，登录 oss 找到 Staging 记录，依次点击Close……等待……Release……再等待一小会儿，你的构建就同步到中央仓库了。

当然别忘了，GPG 签名别丢，更换电脑可以安装 GPG 签名生成步骤生成新的 GPG 公私秘钥；

---

## 常见错误：

### gpg: 签名时失败处理

> gpg: signing failed: Inappropriate ioctl for device

**解决方案**

如果你 macOS，很可能遇到该问题；由于通过 brew 安装的GPG版本是最新版本，存在一些兼容性问题；需要在~/.gnupg目录下增加两个配置文件：

```
echo "use-agent \ npinentry-mode loopback" > gpg.conf
echo "allow-loopback-pinentry" > gpg-agent.conf
```

### 在 OSS 点击 Close 之后出现错误

{% asset_img close-error-0.png [点击 Close 之后出错提示] %}

如上图，构建行首的 Logo 上有一个红色的4，那就说明刚才的 Close 操作，验证没有通过；
点击之后，可以在下面下方的 Activity 中看到详细的错误，如下图：

{% asset_img close-error-1.png [查看错误详情] %}

**解决方案**

其中每一次Close操作都会触发一次校验，校验有很多项，例如签名，POM 文件是否合法，是否包含源代码等等；齿轮是绿色的代表验证通过的项，红色的代表失败的验证项，点击可以查看具体失败的详细信息，对号入座解决问题即可，修改后重新 deploy 上来重复以上操作。

例如上图中，我的 pom 文件中缺少了 Project URL，验证没有通过；

### 执行 Close 出现 Failed: Signature Validation

{% asset_img failed-signature.png [验证构建文件签名失败] %}

**解决方案**

恭喜你，你跟我一样遇到这个麻烦的问题，使用 brew 安装的 GPG 明明上传了公钥，查询查询到了，倒是 sonatype 就是报找不到对应的秘钥；那怎么解决呢；这个应该确认还是没有上传成功的，只是我感觉上传成功了。

这个时候，我是这么解决的，到 https://gpgtools.org/ 上下载了macOS 版本的 GPG Suite Tools客户端工具，安装好之后，打开 GPG Keychain ，这是你可能会直接看到刚才生成的 GPG 秘钥被列出来，你只需要选中，然后右键选中 Send Public to Key Server，接着等待3-5秒，弹出提示 successfully，这一次是真的上传成功了；再次重复 deploy=》Close 操作，你会发现，很快就验证通过了，赶紧去点击 Release 发布吧，谢谢。




