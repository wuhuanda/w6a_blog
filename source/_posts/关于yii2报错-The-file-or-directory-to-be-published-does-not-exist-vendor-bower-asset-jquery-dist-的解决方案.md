---
title: >-
  关于yii2报错: The file or directory to be published does not exist:
  /.../vendor/bower-asset/jquery/dist 的解决方案
category:
- 问题记录
tags:
- php
- yii2
---

# 话不多说，直接上错误截图！

![](https://file.clovemu.com/img/53268a3bc9ed935c81d3377dc0c728aef7eff2.png?imgslim)

# 分析

根据反馈的错误原因，我们能看到它说找不到 /.../vendor/bower-asset/jquery/dist 文件（夹），于是我去到 vendor 里面一番查找果然找不到相关的文件（夹）。

然后我在一番科学搜索后终于 yii 的论坛中找到了解决方案。

# 解决方案

1. 修改 composer.json
```diff
...
"require": {
    "php": ">=5.6.0",
    "yiisoft/yii2": "~2.0.14",
    "yiisoft/yii2-bootstrap": "~2.0.0",
    "yiisoft/yii2-swiftmailer": "~2.0.0 || ~2.1.0",
+   "yidas/yii2-bower-asset": "*"
},
...
"extra": {
    "yii\\composer\\Installer::postCreateProject": {
        "setPermission": [
            {
                "runtime": "0777",
                "web/assets": "0777",
                "yii": "0755"
            }
        ]
    },
    "yii\\composer\\Installer::postInstall": {
        "generateCookieValidationKey": [
            "config/web.php"
        ]
    },
+   "asset-installer-paths": {
+       "npm-asset-library": "vendor/npm",
+       "bower-asset-library": "vendor/bower"
+   }
},
...
```

2. 修改 config/web.php
```diff
'aliases' => [
-   '@bower' => '@vendor/bower-asset',
+   '@bower' => '@vendor/yidas/yii2-bower-asset/bower',
    '@npm'   => '@vendor/npm-asset',
],
```

3. 修改完之后 `composer update` 一下再重新 `php yii serve`，刷新一下浏览器页面看到以下界面就表示成功啦！
![](https://file.clovemu.com/img/b0ca8ae82be691dbbc320108c8653fdedcac22.png?imgslim)

# 参考网址
[composer论坛](https://forum.yiiframework.com/t/composer-2-invalidargumentexception-vendor-bower-asset-jquery-dist/131264/3)