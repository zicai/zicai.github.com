---
layout: post
category : tutorial
title: grunt 使用
tags : [grunt]
---
{% include JB/setup %}
 
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