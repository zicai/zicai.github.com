---
layout: post
category : tutorial
title: Grunt 使用
tags : [grunt]
---

 
1. 生成package.json
 
        npm init

2. 新建Gruntfile.js

        module.exports = function (grunt) {
            grunt.initConfig({
            
            });
        }

3. 文件合并 [https://github.com/gruntjs/grunt-contrib-concat](https://github.com/gruntjs/grunt-contrib-concat)
    
        npm install grunt-contrib-concat --save-dev
    修改Gruntfile.js

        grunt.loadNpmTasks('grunt-contrib-concat');
        grunt.initConfig({
            concat: {
                dist: {
                    files: {
                        'dist/dist.js': ['src/js/lib/*']
                        ]
                    }
                }
            }
        });

4. CSS 压缩 [https://github.com/gruntjs/grunt-contrib-cssmin](https://github.com/gruntjs/grunt-contrib-cssmin)
 
        npm install grunt-contrib-cssmin --save-dev
    修改Gruntfile.js

        grunt.loadNpmTasks('grunt-contrib-cssmin');
        
        grunt.initConfig({
            cssmin: {
            '<%= config.dist %>/ezdo.css': '<%= config.dist %>/.tmp/ezdo.css'
            }
        });
        

5. JS 语法检查 [https://github.com/gruntjs/grunt-contrib-jshint](https://github.com/gruntjs/grunt-contrib-jshint)
    
        npm install grunt-contrib-jshint --save-dev
        npm install --save-dev jshint-stylish  //安装一个reporter
新建.jshintrc文件

        {
        "node": true,
        "browser": true,
        "esnext": true,
        "bitwise": true,
        "camelcase": true,
        "curly": true,
        "eqeqeq": true,
        "immed": true,
        //"indent": 4,
        "latedef": true,
        "newcap": true,
        "noarg": true,
        "quotmark": "single",
        "undef": true,
        "unused": true,
        "strict": true,
        "trailing": true,
        "smarttabs": true,
        "jquery": true
        }

    修改Gruntfile.js
    
        grunt.loadNpmTasks('grunt-contrib-jshint');

        //在 grunt.initConfig 中增加
        jshint: {
            options: {
                jshintrc: '.jshintrc',
                reporter: require('jshint-stylish')
            },
            all: [
                'Gruntfile.js',
                'src/{,*/}*.js'
            ]
        }


6. JS 混淆 [https://github.com/gruntjs/grunt-contrib-uglify](https://github.com/gruntjs/grunt-contrib-uglify)
 
        npm install grunt-contrib-uglify --save-dev
 修改Gruntfile.js

        grunt.loadNpmTasks('grunt-contrib-uglify');
        

7. 文件复制 [https://github.com/gruntjs/grunt-contrib-copy](https://github.com/gruntjs/grunt-contrib-copy)
 
        npm install grunt-contrib-copy --save-dev
        grunt.loadNpmTasks('grunt-contrib-copy');
        

## 其它常用插件        

- [grunt-spritesmith](https://github.com/Ensighten/grunt-spritesmith)

### 编译 sass 有两个选择：

- [grunt-contrib-sass](https://github.com/gruntjs/grunt-contrib-sass) 更稳定的，支持compass，速度稍慢
- [grunt-sass](https://github.com/sindresorhus/grunt-sass) 使用 node-sass 编译 sass

> node-sass 是 Node.js bindings to libsass，the C version of the popular stylesheet preprocessor, Sass。
和原生的Ruby 编译器相比，libsass 更快，但是缺少了一些功能http://sass-compatibility.github.io/。而且也不支持compass




## 调试
`nonull` 如果被设置为 `true`，未匹配的模式也将执行。结合 Grunt 的 `--verbore` 标志, 这个选项可以帮助用来调试文件路径的问题。
