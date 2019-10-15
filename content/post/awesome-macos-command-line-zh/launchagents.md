---
title: "自启动服务"
date: 2019-05-01T14:25:24+08:00
description: ""
hiddenFromHomePage: true
tags: ["Awesome"]
categories: ["Mac"]
---

> 各种有用的自启动服务例子。 [Awesome OS X Command Line](https://github.com/nusr/awesome-macos-command-line-zh) 的一部分
> ``

<!--more-->

## 基本例子

### 定时工作模板

每 5 分钟运行一次。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.example.touchsomefile</string>
    <key>ProgramArguments</key>
    <array>
        <string>touch</string>
        <string>/tmp/helloworld</string>
    </array>
    <key>StartInterval</key>
    <integer>300</integer>
</dict>
</plist>
```

### 按照日历定期工作模板

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.example.touchsomefile</string>
    <key>ProgramArguments</key>
    <array>
        <string>touch</string>
        <string>/tmp/helloworld</string>
    </array>
    <key>StartCalendarInterval</key>
    <dict>
        <key>Minute</key>
        <integer>45</integer>
        <key>Hour</key>
        <integer>13</integer>
        <key>Day</key>
        <integer>7</integer>
    </dict>
</dict>
</plist>
```

### 监控文件夹模板

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.example.watchhostconfig</string>
    <key>ProgramArguments</key>
    <array>
        <string>syslog</string>
        <string>-s</string>
        <string>-l</string>
        <string>notice</string>
        <string>somebody touched /etc/hostconfig</string>
    </array>
    <key>WatchPaths</key>
    <array>
        <string>/etc/hostconfig</string>
    </array>
</dict>
</plist>
```

## Homebrew

### 定时更新 Homebrew

为了使用通知系统，这个功能需要先安装[terminal-notifier](https://github.com/julienXX/terminal-notifier)。可以通过 `brew install terminal-notifier` 安装 terminal-notifier。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.example.homebrew-upgrade</string>
    <key>ProcessType</key>
    <string>Background</string>
    <key>ProgramArguments</key>
    <array>
        <string>/bin/sh</string>
        <string>-c</string>
        <string>/usr/local/bin/brew update &amp;&amp; /usr/local/bin/brew upgrade &amp;&amp; /usr/local/bin/terminal-notifier -title 'Homebrew Upgrader' -message 'Homebrew upgraded!' -appIcon http://cdn.curvve.com/wp-content/uploads/2013/09/homebrew_osx_logo.png</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>StandardErrorPath</key>
    <string>/tmp/com.example.homebrew-upgrade.stderr</string>
    <key>StandardOutPath</key>
    <string>/tmp/com.example.homebrew-upgrade.stdout</string>
    <key>StartCalendarInterval</key>
    <array>
        <dict>
            <key>Hour</key>
            <integer>8</integer>
        </dict>
    </array>
</dict>
</plist>
```


