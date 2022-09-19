---
title: Electron中使用sql.js操作SQLite数据库
excerpt: 如题目
categories:
- 技术文章
tags:
- electron
---

## 关于sql.js
sql.js （https://github.com/kripken/sql.js） 通过使用Emscripten编译SQLite C代码，将SQLite移植到Webassembly。 它使用存储在内存中的虚拟数据库文件，因此不会保留对数据库所做的更改。 但是，它允许您导入任何现有的sqlite文件，并将创建的数据库导出为JavaScript类型的数组。

这里没有C绑定或node-gyp编译，sql.js是一个简单的JavaScript文件，可以像任何传统的JavaScript库一样使用。 如果您正在JavaScript中构建本机应用程序（例如，使用Electron），或者在node.js中工作，则您可能更喜欢使用SQLite与JavaScript的本地绑定 （https://www.npmjs.com/package/sqlite3） 。

SQLite是公共领域，sql.js是MIT许可的。

Sql.js早于WebAssembly，因此从asm.js项目开始。 它仍然支持asm.js以实现向后兼容。

## 为什么要用sql.js
在开发electron应用的时候如果想要使用sqlite3，步骤上除了npm安装以外还要rebuild，比较麻烦，参见electron官方文档之使用 Node 原生模块 （https://electronjs.org/docs/tutorial/using-native-node-modules） 。

如果你想找一个开箱即用的sql库，那么sql.js将是个不错的选择。sql.js是sqlite的Webassembly版，使用上和sqlite基本没有区别。sql.js支持浏览器端直接引入cdn，或者下载sql-wasm.js和sql-wasm.wasm到本地后引入，也支持npm导入。

## 在Electron项目中安装sql.js
1. 假设我们已经搭建好了Electron的一个hello world项目，如 https://xushanxiang.com/2019/11/installing-electron.html
2. 安装sql.js
    ```
    cnpm install sql.js --save
    ```
    安装完后，查看目录结构：
    ```
    tree node_modules/sql.js -L 2
    ```
    ```
    node_modules/sql.js
    ├── AUTHORS
    ├── GUI
    │   └── index.html
    ├── LICENSE
    ├── Makefile
    ├── README.md
    ├── cache
    │   ├── check.txt
    │   ├── extension-functions.c
    │   └── sqlite-amalgamation-3280000.zip
    ├── dist
    │   ├── sql-asm-debug.js
    │   ├── sql-asm-memory-growth.js
    │   ├── sql-asm.js
    │   ├── sql-wasm-debug.js
    │   ├── sql-wasm-debug.wasm
    │   ├── sql-wasm.js
    │   ├── sql-wasm.wasm
    │   ├── worker.sql-asm-debug.js
    │   ├── worker.sql-asm.js
    │   ├── worker.sql-wasm-debug.js
    │   └── worker.sql-wasm.js
    ├── examples
    │   ├── GUI
    │   ├── README.md
    │   ├── persistent.html
    │   ├── repl.html
    │   ├── requireJS.html
    │   ├── simple.html
    │   └── start_local_server.py
    ├── index.html
    ├── out
    │   ├── api.js
    │   ├── extension-functions.bc
    │   ├── sqlite3.bc
    │   └── worker.js
    ├── package.json
    ├── sqlite-src
    │   └── sqlite-amalgamation-3280000
    └── src
        ├── api-data.coffee
        ├── api.coffee
        ├── exported_functions.json
        ├── exported_runtime_methods.json
        ├── exports.coffee
        ├── output-post.js
        ├── output-pre.js
        ├── shell-post.js
        ├── shell-pre.js
        └── worker.coffee
    9 directories, 41 files
    ```

## 在Electron项目中，如何使用sql.js操作SQLite数据库
### 创建一个SQLite数据库
我们可以在命令行中创建，也可以通过在线SQL解释器 （http://kripken.github.io/sql.js/examples/GUI/） 创建并下载一个。

我们使用后者下载一个数据库 sql.db 放到项目根目录。再看一下我们当前的项目文件结构：

```
d tree  -L 1
 .
 ├── index.html
 ├── main.js
 ├── node_modules
 ├── package.json
 └── sql.db
 1 directory, 4 files
```
当然，我们也可以在项目运行时，动态创建一个，不过先创建静态文件目录，拷贝相关文件到静态文件目录。
```
→ mkdir -p static/js
→ cp node_modules/sql.js/dist/sql-wasm.* static/js/
→ ls static/js/
  sql-wasm.js   sql-wasm.wasm
```
再改动 index.html 文件内容：
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>create the database</title>
  </head>
  <body>
    <div id="result"></div>
    <script src='./static/js/sql-wasm.js'></script>
    <!--如果在主进程中，则用var initSqlJs=require('sql-wasm.js')-->
    <script>
      var fs = require("fs");
      var config = {
        // 指定加载sql-wasm.wasm文件的位置
        locateFile: filename => `./static/js/${filename}` 
      };
      initSqlJs(config).then(SQL => {
        // 创建数据库
        var db = new SQL.Database();
        // 运行查询而不读取结果
        db.run("CREATE TABLE test (col1, col2);");
        // 插入两行：(1,111) and (2,222)
        db.run("INSERT INTO test VALUES (?,?), (?,?)", [1,111,2,222]);
        // 将数据库导出到包含SQLite数据库文件的Uint8Array
        // export() 返回值: ( Uint8Array ) — SQLite3数据库文件的字节数组
        var data = db.export();
        // 由于安全性和可用性问题，不建议使用Buffer()和new Buffer()构造函数
        // 改用new Buffer.alloc()、Buffer.allocUnsafe()或Buffer.from()构造方法
        // var buffer = new Buffer(data);
        var buffer = Buffer.from(data, 'binary');
        // 被创建数据库名称
        var filename = "d.sqlite";
        fs.writeFileSync(filename, buffer);
        document.querySelector('#result').innerHTML = filename + "Created successfully.";
      });
    </script>
  </body>
</html>
```
如果你没有安装SQLiteStudio之类的软件，可以把上面动态创建的数据库文件载入 http://kripken.github.io/sql.js/examples/GUI/ ，再执行下面SQL语句得到下面表格结果。
```sql
SELECT * FROM test
```

|   col1   |  col2    |
| ---- | ---- |
|   1   |   111   |
|   2   |   222   |

### 对SQLite数据库进行查询操作
替换 index.html 文件内容如下：
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Execute SELECT sql</title>
  </head>
  <body>
    <div id="result"></div>
    <script src='./static/js/sql-wasm.js'></script>
    <script>
      var fs = require("fs");
      var filebuffer = fs.readFileSync('d.sqlite');
      var config = {
        // 指定加载sql-wasm.wasm文件的位置
        locateFile: filename => `./static/js/${filename}` 
      };
      initSqlJs(config).then(SQL => {
        // 加载数据库到内存中
        var db = new SQL.Database(filebuffer);
        var res = db.exec("SELECT * FROM test");
        // Returns: ( Array<QueryResults> ) — An array of results.
        console.log(JSON.stringify(res));
        // [{"columns":["col1","col2"],"values":[[1,111],[2,222]]}]
      });
    </script>
  </body>
</html>
```

### 对SQLite数据库进行增、改、删操作
替换 index.html 文件内容如下：
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Execute SELECT sql</title>
  </head>
  <body>
    <div id="result"></div>
    <button onclick="delete_row(3)">删除id为3的记录</button>
    <script src='./static/js/sql-wasm.js'></script>
    <script>
      var fs = require("fs");
      var filebuffer = fs.readFileSync('d.sqlite');
      var db;
      var config = {
        // 指定加载sql-wasm.wasm文件的位置
        locateFile: filename => `./static/js/${filename}` 
      };
      initSqlJs(config).then(SQL => {
        // 加载数据库到内存中
        db = new SQL.Database(filebuffer);

        // 插入两行：(1,111) and (2,222)
        db.run("INSERT INTO test VALUES (?,?), (?,?)", [3,333,4,444]);
        var res = db.exec("SELECT * FROM test");
        console.log(JSON.stringify(res));
        // [{"columns":["col1","col2"],"values":[[1,111],[2,222],[3,333],[4,444]]}]

        // 修改一行
        db.run("UPDATE test SET col2=(?) WHERE col1=(?)", [3333,3]);
        res = db.exec("SELECT * FROM test");
        console.log(JSON.stringify(res));
        // [{"columns":["col1","col2"],"values":[[1,111],[2,222],[3,3333],[4,444]]}]

        // 删除一行
        db.run("DELETE FROM test WHERE col1=(?)", [4]);
        res = db.exec("SELECT * FROM test");
        console.log(db.getRowsModified());
        console.log(JSON.stringify(res));
        // [{"columns":["col1","col2"],"values":[[1,111],[2,222],[3,3333]]}]

        // 执行一条sql语句，并为结果的每一行调用一个回调
        db.each("SELECT * FROM test where col1<$col1",
          {$col1:3},
          function(row){console.log(row.col2);}
        );

        // 注意：上面的增、改、删都只是对载入内存中的数据进行变更，我们需要把内存中的数据再存入磁盘文件中，不然一切皆枉然。
        var data = db.export();
        var buffer = Buffer.from(data, 'binary');
        fs.writeFileSync("d.sqlite", buffer);
        document.querySelector('#result').innerHTML = "SQLite database changed.";
        // 完成操作后，必须关闭数据库，与数据库关联的内存和所有关联的语句将被释放，否则内存消耗将永远增加
        // db.close();
      });
      function delete_row(id){
        db.run("DELETE FROM test WHERE col1=(?)", [id]);
        res = db.exec("SELECT * FROM test");
        console.log(db.getRowsModified());
        console.log(JSON.stringify(res));
      }
    </script>
  </body>
</html>
```

## 参考链接
[Electron中使用sql.js操作SQLite数据库](https://www.cnblogs.com/alex96/p/12252992.html)