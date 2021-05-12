---
layout: post
title: "my framework 之注册自加载函数"
date: 2021-05-12 15:07
comments: true
reward: false
top: false
brief: ""
tags: 
	- 框架自动加载模块
key: "framework autoload"
---

### 实现框架注册自加载

> 使用spl_autoload_register函数注册自加载函数到__autoload队列中，配合使用命名空间，当使用一个类的时候可以自动载入(require)类文件。注册完成自加载逻辑后，我们就可以使用use和配合命名空间申明对某个类文件的依赖。

> 来一张不知道画的啥的图搞清楚执行流程
![avatar](assets/blogImg/framework_20210512.jpg)

> 不说屁话了，直接上代码实现。其实代码是参考easy-php框架源码。特地拿出来拆讲理解。

[file: public/index.php]
``` 
//载入框架运行文件
require(dirname(__DIR__) . DIRECTORY_SEPARATOR . 'framework/run.php');
```

[file: framework/run.php]
```
require(__DIR__ . '/App.php');
/**
 * 初始化应用
 */
$app = new Framework\App(realpath(__DIR__ . '/..'), function () {
    return require(__DIR__ . '/Load.php');
});
```

[file: framework/App.php]
```
namespace Framework;
use Closure;
class App
{
    private $rootPath;
    public static $app;

    public function __construct($rootPath, Closure $loader) 
    {
        // 根目录
        $this->rootPath = $rootPath;
        // 注册自加载
        $loader();
        Load::register($this);
        self::$app = $this;
    }
}
```

[file: framework/Load.php]
```
namespace Framework;
use Framework\App;
class Load
{
    /**
     * 类名映射
     * 
     * @var array
     */
    public static $map = [];

    /**
     * 类命名空间映射
     * 
     * @var array
     */
    public static $namespaceMap = [];

    public static function register(App $app)
    {
        self::$namespaceMap = [
            'Framework' => $app->rootPath
        ];
        //注册框架加载函数
        spl_autoload_register(['Framework\Load', 'autoload']);
    }

    static public function autoload($class)
    {
        $classOrigin = $class;
        $classInfo = explode('\\', $class);
        $className = array_pop($classInfo);
        foreach ($classInfo as &$v) {
            $v = strtolower($v);
        }
        unset($v);
        array_push($className, $className);
        $class = implode('\\', $classInfo);
        $path = self::$namespaceMap['Framework'];
        $classPath = $path . '/' . str_replace('\\', '/', $class) . '.php';
        self::$map[$classOrigin] = $classPath;
        require $classPath;
    }
}
```