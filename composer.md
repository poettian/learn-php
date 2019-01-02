composer
=======

### 自动加载 

每个项目或包中都有相应的 composer.json 文件，其中的配置项 `autoload` 和 `autoload-dev` 定义了自动加载的规则，包括 psr-0、psr-4、file 以及 classmap，在执行 `composer install` 或 `composer dump-autoload` 时，composer会自动更新 **vendor/composer/** 目录下的 autoload_* 文件，从而实现包的自动加载。 

`composer dump-autoload` 和 `composer dump-autoload -o` 的区别是：前者只是根据配置项更新相应的 autoload_* 文件内容，而后者还会自动检索配置项中命名空间对应的目录下的 class 更新到 classmap 中。 

有一点要注意，配置项中命名空间对应的目录，在项目中是以根路径开始，在包中是以包路径开始的。 

### lock file 

包中的 lock file 可以提交也可以不提交，这不会影响项目。因为项目在安装包时生成的 lock file 中已经记录了包依赖的版本，并不会参考包中的 lock file。当然如果你在包的根目录下执行 `composer install`，是会根据 lock file 来执行版本约束的。 

### 仓库 

composer 定义的仓库有几个种类： 

##### Composer 

composer 默认的仓库类型，packagist 就是使用的这种类型。主要靠一个文件 `packages.json` 来定义所有的元数据，可以查看 

[packages.json](https://packagist.org/packages.json) 

[provider-includes](https://packagist.org/p/provider-latest$14f62fee2179709d9d0a8bc14bacf3b05825734e217378f57a2b9064891a857d.json) 

[providers-url](https://packagist.org/p/poettian/hello-world$001170bc006de2edc5c10a75331ee425c4170a0bc2487cc2f781aa5ff17327ac.json) 

加深理解。 

##### VCS 

一般是用来修改原始包用的，从而让你可以选择使用你自己fork来并修改后的包 

```json 

{ 

	"repositories": [ 

		{ 

            "type": "vcs", 

            "url": "https://github.com/igorw/monolog" 

        } 

	], 

	"require": { 

		"monolog/monolog": "dev-bugfix" 

	} 

} 

```

##### PEAR 

##### Package 

### 版本约束 

##### 使用标签 

当在版本约束中指定了某个具体数字版本或者一个数字范围版本时，composer 内部会维护一个版本列表，这个版本列表是从vcs中请求tag list得到的。根据这份列表，composer 会从中找到符合条件的最高版本（一般情况下），然后下载对应的zip release包，然后解压到vendor目录中去。 

##### 使用分支 

如果要使用分支，需要指定 `dev-*`（有时是后缀）前缀。composer 会克隆相应的仓库到 vendor 目录，而 tag 只是 copy 文件。如果分支使用了 v1 这种类似于标签的命名，那么就需要指定版本约束为：v1.x-dev。 

##### minimum-stability 

composer 默认取 stable 版本的包，所以如果要用 v1.1-BETA v1.1-RC1 v1.1-RC2 这样的版本就必须显示指明。 

可以在 composer.json 的配置项 `minimum-stability` 或者也可以在版本约束中使用 "stability flags"（比如："monolog/monolog": "1.0.\*@beta"，可选项有 dev, alpha, beta, RC, 和 stable）。 

### 杂项 

##### replace 

主要为了解决重复依赖替换的问题： 

1. `A 依赖 B，A 依赖 C，C 依赖 B`，假设自己 fork 修改了 B 为新命名的 D，那么 C 也应该依赖新的 D，这时就可以在 D 的 composer.json 中使用 replace 声明对 B 的替换： 

```json 

"replace": { 

	"B":"1.*" 

} 

```

2. 参考 laravel/framework 包，这是一个主包，里面有很多子包，当我们依赖并引入 laravel/framework 包时，子包也一并被引入，这时如果其他包有对子包的依赖，完全可以使用主包所包含的子包，而不用重新引入一个子包到 vendor 目录下，这时就可以在主包的 composer.json 中使用 replace 标明： 

```json 

"replace": { 

	"illuminate/auth": "self.version" 

} 

```

self.version 表示主包的版本。 

可以参考： 

https://jaceju.net/2018-11-02-composer-replace/ 



##### 关于require-dev packages的安装

composer 只会安装项目中主 composer.json 文件中 require-dev packages，项目依赖包中 composer.json 里的 require-dev packages 不会安装。

这主要是因为依赖包中 require-dev 是供开发依赖包使用的，而我们开发项目时只是使用依赖包但不需要开发它们。