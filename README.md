
# require.js的基本使用

## 基本的目录组织方式

* index.html  
* resource
    * app
        * hd.js
        * util.js
    * css
        * bootstrap.min.css
        * font-awesome.css
    * font
    * js
        * jquery.min.js
    * lib
        * angular.min.js
        * bootstrap.min.js
        * css.min.js
    * main.js
    * require.js


## 解决页面加载是，加载模块失败的3种解决办法


```js

// mian.js
require.config({
    baseUrl: '../resource/app',
    paths: {
        'hd': 'hd',
        'css': '../lib/css.min',
        'jquery': '../lib/jquery.min',
        'angular': '../lib/angular.min',
        'bootstrap': '../lib/bootstrap.min',
    }
});

// html
<script data-main="../resource/main" src="../resource/require.js"></script>
// 配置文件是异步进行设置的，即加载完require.js之后，下面的代码已经开始走了，即解析data-main的过程与执行下面代码的过程是同步进行的，说白了就是，下面的代码在引包的时候，根本就没有读到配置项main.js中的内容，因为二者是同时进行的，配置项还没有执行完，下面代码yijing 开始执行了；
<script>
    require(['jquery', 'angular'], function ($, angular) {
        $('body').css({'backgroundColor': 'red'});
    })
</script>

```

> 解决方式一：引包的时候，将路径补全，当配置项成功运行的时候，只需要将名字写在里面就行了，而在配置项没有运行的时候，将路径补全;  还需要将回调函数中的参数删除，因为此时系统并非是走模块的，所以系统没有办法将，jquery对象与angular对象赋值给参数；

```js
<script>
    require(['lib/jquery.min', 'lib/angular.min'], function () {
        $('body').css({'backgroundColor': 'red'});
    })
</script>

```

> 解决方式二： 将引包函数放到配置项函数代码下面；其实只需要保证配置项先于引包操作执行就可以了；

```js
// mian.js
require.config({
    baseUrl: '../resource/app',
    paths: {
        'hd': 'hd',
        'css': '../lib/css.min',
        'jquery': '../lib/jquery.min',
        'angular': '../lib/angular.min',
        'bootstrap': '../lib/bootstrap.min',
    }
});
// 将引包操作，放到配置项代码的后面；
require(['lib/jquery.min', 'lib/angular.min'], function () {
    $('body').css({'backgroundColor': 'red'});
})
```

> 解决方式三: 将require.js与配置文件main.js分别写在两个script标签中，并且保证两者的先后执行顺序；

```js
<script src="../resource/require.js"></script>
<script src="../resource/main.js"></script>
<script>
    require(['jquery', 'angular'], function ($, angular) {
        $('body').css({'backgroundColor': 'red'});
    })
</script>
```

## 使用define去自定义模块

* 定义模块

 ```js
//  [] 表示该自定义模块所依赖的其它模块；
define(['jquery'], function () {
    return {
        change: function () {
            $('body').css({'backgroundColor': 'red'});
        },
        show: function () {
            alert('后盾人')
        },
        message: function () {
            alert('houdunren.com')
        }
    }
});

// define([], function () {
//     return {
//         change: function () {
//             require(['jquery'],funciton($){
//                 $('body').css({'backgroundColor': 'red'});
//             })
//         },
//         show: function () {
//             alert('后盾人')
//         },
//         message: function () {
//             alert('houdunren.com')
//         }
//     }
// });


 ```

 * 修改配置项

```js
    //自定义的模块都是放在resource目录下的app目录下面
    // 由于自定义的模块放在该目录，所以改目录的内容时常会变动，就将该路径作为基准路径，
    baseUrl: '../resource/app',
    //下面引用的这些包，基本上不怎么会动，就直接写死就可以了； 注意路径是相对与基准路径来的；
    paths: {
        'hd': 'hd',
        'css': '../lib/css.min',
        'jquery': '../lib/jquery.min',
        'angular': '../lib/angular.min',
        'bootstrap': '../lib/bootstrap.min',
    },

```

* 引包

```js
<script src="../resource/require.js"></script>
<script src="../resource/main.js"></script>
<script>
    //此处的原理就是，使用define定义的模块引用返回之后，会将返回的对象return{} 赋值给回调函数的参数，从而参数可以调用相应的方法； 
    require(['util'],function(b){
        b.change();
    })
</script>
```

## 多个模块之间的依赖关系实例讲解

```js
require.config({
    // 定义基准路径
    baseUrl: '../resource/app',
    // 定义模块路径
    paths: {
        'hd': 'hd',
        // css库 用来加载css文件使用的；
        'css': '../lib/css.min',
        'jquery': '../lib/jquery.min',
        'angular': '../lib/angular.min',
        'bootstrap': '../lib/bootstrap.min',
    },
    // 定义依赖关系
    shim: {
        'hd': {
            // exports: 'modal',
            init: function () {
                return {
                    modal: modal,
                    success: success,
                }
            }
        },

        // 表示针对bootsrap这个模块的依赖关系；
        'bootstrap': {
            // 要引入css文件，需要首先引入css.min.js这个专用库；font-awesome.min.css是一个字体库；
            'deps': ['jquery', 'css!../css/bootstrap.min.css', 
            'css!../css/font-awesome.min.css']
        }
    }
});

```


## 处理非标准化模块

```js
// html
<script src="../resource/require.js"></script>
<script src="../resource/main.js"></script>
<script>
    require(['hd'],function(f){
        f.modal();
    })
</script>

// 主配置文件main.js
require.config({
    baseUrl: '../resource/app',
    paths: {
        'hd': 'hd',
        'css': '../lib/css.min',
        'jquery': '../lib/jquery.min',
        'angular': '../lib/angular.min',
        'bootstrap': '../lib/bootstrap.min',
    },
    shim: {
        'hd': {
            // 若hd.js中只有一个方法，则去采用exports的写法；
            // exports: 'modal',

            // hd.js中存有多个方法，则采用init的写法，当exports与init方法同时存在的时候，init方法的优先级要高；
            init: function () {
                return {
                    modal: modal,
                    success: success,
                }
            }
        },
        //houdunren.com
        'bootstrap': {
            'deps': ['jquery', 'css!../css/bootstrap.min.css', 'css!../css/font-awesome.min.css']
        }
    }
});

// hd.js 内部使用的并非是标准的define写法，而是利用传统的function的形式；
function modal() {
    alert('后盾人 modal');
}
function success() {
    alert('后盾人 success');
}
```