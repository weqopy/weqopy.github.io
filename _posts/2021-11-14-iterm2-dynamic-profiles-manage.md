---
layout: post
title: iTerm2 dynamic profiles manage
date: 2021-11-14 09:28:25 +0800
category: Personal
tag: Tool
---

* content
{: toc}
---

## 目的

本文为整理当前使用`iTerm2`管理多个服务器的使用经验而作。

## 关键字

- iTerm2
- expect
- Dynamic Profile

## 正文

### `iTerm2` 连接服务器

一般而言，使用终端连接服务器时，直接通过命令行输入`ssh user@host`即可，但是当涉及管理多个服务器、并且可能需要经常临时连接时，常有捉襟见肘之感。

此时进一步的思路是使用ssh密钥，但在笔者的使用场景中，该方法也有不便之处：其一，部分服务器因特殊原因不便使用ssh密钥；其二，当服务器数量过多时，无法搜索、只能凭记忆输入ssh别名也比较痛苦；其三，部分服务器需要使用中间服务器作跳板机进行连接，到达跳板机时，仍需要再另行操作连接目标服务器，不太流畅。

当前笔者使用的方式为`iTerm2`自带的`Profiles`功能，结合`expect`方式进行快速连接，并通过`Dynamic Profiles`配置`Profile`模板，以下进行介绍。

### `Profiles` 配置管理多服务器

`iTerm2`自带管理`Profiles`功能，简单配置`name`和`command`即可

![profiles](/assets/profiles.png)

此时可通过快捷键`cmd+o`打开`iTerm2`的`Open Profiles`弹窗，查看`Profiles`列表并通过选择、搜索等方式选中并连接服务器

![open_profiles](/assets/open_profiles.png)

当使用熟悉后，更快速一些的方式是，通过快捷键`cmd+shift+o`打开`iTerm2`的`Open Quickly`弹窗，并通过模糊搜索`Profile Name`连接服务器

![open_quickly](/assets/open_quickly.png)

### expect 连接服务器

一个简单的示例文件：
```shell
#!/usr/bin/expect -f
set user USER
set host HOST
set password PASSWORD
set port PORT
set dir DIR

spawn ssh -o ServerAliveInterval=30 -p$port $user@$host
expect "*assword:*"
send "$password\r"

# after login
expect "*\$*"
send "bash\r"
send "cd $dir\r"
send "ls"
interact
expect eof
```

将该文件保存到本地目录，如`～/host_example.sh`，然后通过`expect ～/host_example.sh`命令，连接服务器，并进入`bash`环境，进入`DIR`目录，输入`ls`查看当前目录文件

对于需要跳板机的情况，在上述示例文件的基础上继续添加`send`命令即可
```shell
# set 设置省略，参考上方文件


# after login
expect "*\$*"
send "bash\r"

# 暂停留出冗余时间后，连接目标服务器
# port2、user2等参考上方set配置
sleep 2
send "ssh -p$port2 $user2@$host2\r"
expect "*assword:*"
send "$password2\r"
expect "*\$*"
send "bash\r"
send "cd $dir\r"
send "ls"
interact
expect eof
```

### Profiles结合expect

`Profiles`方便管理多服务器，结合`expect`进行更复杂的逻辑处理，在以上内容的基础上，直接使用`expect`命令替换`Profile`中的`command`命令即可

![profile_expect](/assets/profile_expect.png)

### Dynamic Profiles

当管理服务器数量增加时，可考虑配置`Profile`模板，使用`iTerm2`的`Dynamic Profiles`可以比较方便的进行管理。

1. 配置`Profile`模板，建议模板`Name`特殊标记以便识别，同时后续真实服务器`Name`进行区分，以免在通过快捷键连接服务器时对搜索结果产生干扰
2. 进入`iTerm2`目录`~/Library/Application Support/iTerm2/DynamicProfiles/`
3. 新建文件，这里以笔者使用的`json`文件为例
```json
{
    "Profiles": [
        {
            "Name": "host1",
            "Guid": "host1",
            "Command": "expect \/Users\/macname\/host1.sh",
            "Dynamic Profile Parent Name": "host_example"
        },
        {
            "Name": "host2",
            "Guid": "host2",
            "Command": "expect \/Users\/macname\/host2.sh",
            "Dynamic Profile Parent Name": "host_example"
        },
```

> `Guid`不能重复，需要保证唯一性
>
> `host1.sh`, `host2.sh`为[expect 连接服务器](#expect-连接服务器)中配置的文件

此时在`Profiles`中会自动出现两个新的`Profile`，再通过`Open Quickly`即可快速连接，后续可通过调整模板配置统一调整多个服务器配置，如配色、文字等。

![dynamic_profile](/assets/dynamic_profile.png)

---

## 参考

[expect-提供用户名和密码的ssh自动登录](https://www.cxyzjd.com/article/gyxinguan/75089041)

[iTerm2 Documentation-Dynamic Profiles](https://iterm2.com/documentation-dynamic-profiles.html)
