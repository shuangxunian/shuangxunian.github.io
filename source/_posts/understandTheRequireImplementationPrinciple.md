---
title: 彻底搞懂 Node.js 中的 Require 机制(源码分析到手写实践)
excerpt: 关于require的实现原理
date: 2022-09-30
categories:
- 技术文章
tags:
- js
---

## 前言
本文你能学到：
1. 自己手写实现一个 require，面试用也可以。
2. 如何看 Node.js 源码
3. require 函数是如何产生的？为什么在 module 中可以直接使用。
4. require 加载原生模块时候如何处理的，为什么 require('net') 可以直接找到
5. Node.js 中 require 会出现循环引用问题吗？
6. require 是同步还是异步的？为什么？
7. exports 和 module.exports 的区别是什么？
8. 你知道 require 加载的过程中使用了 vm 模块吗？vm 模块是做什么的？vm 模块除了 require 源码用到还有哪些应用场景。

## 什么是 CommonJS
每一个文件就是一个模块，拥有自己独立的作用域，变量，以及方法等，对其他的模块都不可见。CommonJS 规范规定，每个模块内部，module 变量代表当前模块。这个变量是一个对象，它的exports属性（即module.exports）是对外的接口。

## 模块分类
- 原生(核心)模块：Node 提供的模块我们都称之为原生模块
    - 内建模块:Node.js 原生提供的模块中，由纯 C/C++ 编写的称为内建模块
    - 全局模块：Node.js在启动时，会生成一个全局量 process
    - 除了上面两种可以直接 require 的所有原生模块
- 文件模块：用户编写的模块
    - 普通文件模块：node_modules 下面的模块，或者我们自己开发时候写的每个文件内容。
    - C++ 扩展模块：用户自己编写的 C++ 扩展模块或者第三方 C++ 扩展模块

## 模块加载
介绍了上面的模块分类，正常应该到介绍不同模块的加载环节，这里不这样,只列出目录。先带你看一遍源码，再手写一下，然后我想你自己总结一下这几种模块的加载区别。
**加载 Node.js 原生模块**
本文不包括直接调用内建纯C/C++模块，也不推荐这样使用，因为我们正常调用的原生模块都是通过 js封装一层，它们自己再去调用，你想直接调用的 Node.js提供的存C/C++ 内建模块，js 封装的一层也都能做到。那部分内容放在 Node.js与 C++ 那些事的文章中介绍。
**require 加载普通文件模块**
**require 加载 C++ 扩展文件模块**

## require 加载原理(源码分析与手写)
require 源码并不复杂，这里采用的是边看源码边手写的方式讲解(我们最终实现的require 是简易版本，一些源码提到，但是简易版本不会实现)，实现 require 其实就是实现整个 Node.js 的模块加载机制，Node.js 的模块加载机制总结下来一共八步。

### 1.基础准备阶段
Node.js 模块加载的主流程都在 Module 类中，在源码的 https://github.com/nodejs/node/blob/master/lib/internal/modules/cjs/loader.js#L150 中进行了基础 Module 类定义，这个构造函数中的内容主要做一些值的初始化，我们自己对照着实现下，为了和源码有一个区别，本文使用 KoalaModule 命名。
```javascript
function KoalaModule(id = '') {
    this.id = id;       // 这个id其实就是我们require的路径
    this.path = path.dirname(id);     // path是Node.js内置模块，用它来获取传入参数对应的文件夹路径
    this.exports = {};        // 导出的东西放这里，初始化为空对象
    this.filename = null;     // 模块对应的文件名
    this.loaded = false;      // loaded用来标识当前模块是否已经加载
}

KoalaModule._cache = Object.create(null); //创建一个空的缓存对象
KoalaModule._extensions = Object.create(null); // 创建一个空的扩展点名类型函数对象(后面会知道用来做什么)
```
然后在源码中你会找到 require 函数,在 KoalaModule 的原型链上，我们实现下
```javascript
Module.prototype.require = function(id) {
    return Module._load(id, this, /* isMain */ false);
};
```
在源码中你会发现又调用了_load函数，找到源码中的 _load 函数，(源码位置：https://github.com/nodejs/node/blob/master/lib/internal/modules/cjs/loader.js#L724) 下面的所有步骤都是在这个函数中完成调用和 return的，实现简易版_load函数。
```javascript
KoalaModule._load = function (request) {    // request是我们传入的路劲参数
// 2.路径分析并定位到文件
    const filename = KoalaModule._resolveFilename(request);

    // 3.判断模块是否加载过(缓存判断)
    const cachedModule = koalaModule._cache[filename];
    if (cachedModule !== undefined) {
        return cachedModule.exports;
    }
    // 4. 去加载 node 原生模块中
    /*const mod = loadNativeModule(filename, request);
    if (mod && mod.canBeRequiredByUsers) return mod.exports;*/

    // 5. 如果缓存不存在，我们需自行加载模块，new 一个 KoalaModule实例
    // 加载完成直接返回module.exports
    const module = new KoalaModule(filename);

    // 6. load加载之前加入缓存，这也是不会造成循环引用问题的原因，但是循环引用，这个缓存里面的exports可能还没有或者不完整
    KoalaModule._cache[filename] = module;
    // 7. module.load 真正的去加载代码
    module.load(filename);
    // 8. 返回模块的module.exports 
    return module.exports;
}
```
这个函数的源码中有一些其他逻辑的细节判断，有兴趣的小伙伴再学习下，我提出了核心主干。

### 2.路径分析并定位到文件
找到源码中的 _resolveFilename 函数，这个方法是通过用户传入的require参数来解析到真正的文件地址。(源码地址：https://github.com/nodejs/node/blob/master/lib/internal/modules/cjs/loader.js#L816)
这个函数源码中比较复杂，因为 require传递过来的值需要一层一层的判断，同时支持多种参数：内置模块，相对路径，绝对路径，文件夹和第三方模块等等，如果是文件夹或者第三方模块还要解析里面的 package.json 和 index.js。这里简单处理，只实现通过相对路径和绝对路径来查找文件，并支持判断文件js和json后缀名判断:
```javascript
KoalaModule._resolveFilename = function (request) {
    const filename = path.resolve(request);   // 获取传入参数对应的绝对路径
    const extname = path.extname(request);    // 获取文件后缀名
    // 如果没有文件后缀名，判断是否可以添加.js和.json
    if (!extname) {
        const exts = Object.keys(KoalaModule._extensions);
        for (let i = 0; i < exts.length; i++) {
            const currentPath = `${filename}${exts[i]}`;
            // 如果拼接后的文件存在，返回拼接的路径
            if (fs.existsSync(currentPath)) {
            return currentPath;
            }
        }
    }
    return filename;
}
```

### 3.判断模块是否加载过(缓存判断)
判断这个找到的模块文件是否缓存过，如果缓存过，直接返回 cachedModule.exports, 这里就会想到一个问题为什么在 Node.js 中模块重复引用也不会又性能问题，因为做了缓存。(源码位置：https://github.com/nodejs/node/blob/master/lib/internal/modules/cjs/loader.js#L747)
```javascript
const cachedModule = koalaModule._cache[filename];
if (cachedModule !== undefined) {
    return cachedModule.exports;
}
```
### 4.去加载 node 原生模块
如果没有进行缓存过，会调用一个加载原生模块的函数，这里分析一下。
**注意：**第四部分代码我们没有进行手写实现，在_load中进行了注释，但是这里进行了一遍分析，我们写的代码是如何调用到原生模块，本部分涉及到你可能会不想看的C内容，其实也可以忽略掉，过一遍就能知道最后的结论为什么是那样了，不然看了书记不住为什么这样。
比如我们require(net),走完前面的缓存判断就会到达这个loadNativeModule函数(源码位置:https://github.com/nodejs/node/blob/802c98d65de40f245781f591a0b3b38336d1af94/lib/internal/modules/cjs/helpers.js#L31)
```javascript
const mod = loadNativeModule(filename, request);
if (mod && mod.canBeRequiredByUsers) return mod.exports;
```
继续往下看 loadNativeModule 函数的调用。
```javascript
function loadNativeModule(filename, request) {
    // 这里判断下是不是在原生js模块中  ，NativeModule在bootstrap/loader.js中定义
    const mod = NativeModule.map.get(filename);
    if (mod) {
        debug('load native module %s', request);
        mod.compileForPublicLoader();
        return mod;
    }
}
```
mod 是一个 NativeModule 对象，这个对象很常见，在 node启动一个文件时候也会用到(源码位置：https://github.com/nodejs/node/blob/802c98d65de40f245781f591a0b3b38336d1af94/lib/internal/bootstrap/loaders.js#L161)
然后到了 mod 的核心函数mod.compileForPublicLoader();,下面这段代码相对源码删除了一些非核心部分。
```javascript
compileForPublicLoader() {  
    this.compileForInternalLoader();  
    return this.exports;  
}  

compileForInternalLoader() {  
    if (this.loaded || this.loading) {  
        return this.exports;  
    }  
    // id就是我们要加载的模块，比如net 
    const id = this.id;  
    this.loading = true;  
    try {  
        const fn = compileFunction(id);  
        fn(this.exports, nativeModuleRequire, this, process, internalBinding, primordials);  
        this.loaded = true;  
    } finally {  
        this.loading = false;  
    }
    return this.exports;  
}
```
我们重点看compileFunction这里的逻辑。该函数是node_native_module_env.cc模块导出的函数。具体的代码就不贴了，通过层层查找，最后到 node_native_module.cc 的NativeModuleLoader::CompileAsModule。
```c++
MaybeLocal<Function> NativeModuleLoader::CompileAsModule(  
Local<Context> context,  
const char* id,  
NativeModuleLoader::Result* result) {  
    Isolate* isolate = context->GetIsolate();  
    // 函数的形参  
    std::vector<Local<String>> parameters = {  
        FIXED_ONE_BYTE_STRING(isolate, "exports"),  
        FIXED_ONE_BYTE_STRING(isolate, "require"),  
        FIXED_ONE_BYTE_STRING(isolate, "module"),  
        FIXED_ONE_BYTE_STRING(isolate, "process"),  
        FIXED_ONE_BYTE_STRING(isolate, "internalBinding"),  
        FIXED_ONE_BYTE_STRING(isolate, "primordials")
    };  
    // 编译出一个函数  
    return LookupAndCompile(context, id, &parameters, result);  
}
```
我们继续看LookupAndCompile。
```c++
MaybeLocal<Function> NativeModuleLoader::LookupAndCompile(
    Local<Context> context,
    const char* id,
    std::vector<Local<String> >* parameters,
    NativeModuleLoader::Result* result ){
    Isolate			* isolate = context->GetIsolate();
    EscapableHandleScope	scope( isolate );

    Local<String> source;
    /* 找到原生js模块的地址 */
    if ( !LoadBuiltinModuleSource( isolate, id ).ToLocal( &source ) )
    {
        return({});
    }
    /* ‘net’ + ‘.js’ */
    std::string	filename_s	= id + std::string( ".js" );
    Local<String>	filename	=
        OneByteString( isolate, filename_s.c_str(), filename_s.size() );
    /*
    * 省略一些参数处理
    * 脚本源码
    */
    ScriptCompiler::Source script_source( source, origin, cached_data );
    /* 编译出一个函数 */
    MaybeLocal<Function> maybe_fun =
        ScriptCompiler::CompileFunctionInContext( context,
                            &script_source,
                            parameters->size(),
                            parameters->data(),
                            0,
                            nullptr,
                            options );
    Local<Function> fun = maybe_fun.ToLocalChecked();
    return(scope.Escape( fun ) );
}
```
LookupAndCompile 函数首先找到加载模块的源码，然后编译出一个函数。我们看一下LoadBuiltinModuleSource 如何查找模块源码的。
```c++
MaybeLocal<String> NativeModuleLoader::LoadBuiltinModuleSource(Isolate* isolate, const char* id) {  
    const auto source_it = source_.find(id);  
    return source_it->second.ToStringChecked(isolate);  
}
```
这里是 id 是 net，通过该 id 从 _source 中找到对应的数据，那么_source 是什么呢？Nodejs 为了提高效率，把原生 js 模块的源码字符串直接转成 ascii 码存到**内存**里。这样加载这些模块的时候，就不需要硬盘 io 了。直接从内存读取就行。_source 的定义（在 node_javascript.cc里，负责编译nodejs 源码或者执行 js2c.py 生成）。
结论：Node.js 在启动时候直接从内存中读取内容，我们通过 require 加载 net 原生模块时，通过 NativeModule的compileForInternalLoader，最终会在 _source 中找到对应的源码字符串，然后编译成一个函数，然后去执行这个函数,执行函数的时候传递 nativeModuleRequire和internalBinding两个函数，nativeModuleRequire用于加载原生 js 模块，internalBinding用于加载纯C++ 编写的内置模块。
```javascript
const fn = compileFunction(id);  
fn(this.exports, nativeModuleRequire, this, process, internalBinding, primordials);
```

### 5.创建一个 KoalaModule 实例
如果不是原生 node 模块，就会当作普通文件模块加载，自己创建一个 KoalaModule 实例，去完成加载。
```javascript
const module = new KoalaModule(filename);
```

### 6.添加缓存
我把这一小步骤单独提出的原因，想说明的是先进行缓存的添加，然后进行的模块代码的加载，这样就会出现下面的结论，Node.js 官网也有单独介绍,可以自己试一下。
1. main 加载a，a 在真正加载前先去缓存中占一个位置
2. a 在正式加载时加载了 b
3. b 又去加载了 a，这时候缓存中已经有 a 了，所以直接返回 a.exports，这时候 exports 很有可能是不完整的内容。

### 7.module.load 真正的去加载代码
不在缓存，不是原生模块，缓存已经添加完，我们通过这个 load 函数去加载文件模块，源码中位置(https://github.com/nodejs/node/blob/master/lib/internal/modules/cjs/loader.js#L936)
源码中，会有一个获取文件扩展名的函数findLongestRegisteredExtension 这个方法的具体内容是有扩展名的取扩展名，没有的都按照 .js 为扩展名处理，在这个之前会判断一下 Module._extesions 支持的扩展名，不是所有都支持。
我们自己实现一下 load 函数。
```javascript
KoalaModule.prototype.load = function (filename) {
    // 获取文件后缀名(我们忽略掉了findLongestRegisteredExtension函数，有兴趣小伙伴自己实现)
    const extname = path.extname(filename);
    // 根据不同的后缀名去进行不同的处理
    KoalaModule._extensions[extname](this, filename);
    this.loaded = true;
}
```
获取到扩展名之后，根据不通的扩展名去执行不同的扩展名函数，源码中支持的扩展名有.js,.json,.node;
1. 加载.js
    定位到加载 .js 的源码位置 (https://github.com/nodejs/node/blob/master/lib/internal/modules/cjs/loader.js#L1092)
    我们自己实现一下执行.js代码。
    ```javascript
    KoalaModule._extensions['.js'] = function (module, filename) {
        const content = fs.readFileSync(filename, 'utf8');
        module._compile(content, filename);
    }
    ```
    KoalaModule._extensions 中 _compile 函数的执行。找到对应的源码位置 (https://github.com/nodejs/node/blob/master/lib/internal/modules/cjs/loader.js#L1037) ，源码中这里还使用 proxy，我们进行一下简单实现。
    ```javascript
    KoalaModule.wrapper = [
    '(function (exports, require, module, __filename, __dirname) { ',
    '\n});'
    ];

    KoalaModule.wrap = function (script) {
    return KoalaModule.wrapper[0] + script + KoalaModule.wrapper[1];
    };

    KoalaModule.prototype._compile = function (content, filename) {
        const wrapper = KoalaModule.wrap(content);    // 获取包装后函数体

        // vm是 Node.js 的虚拟机模块，runInThisContext方法可以接受一个字符串并将它转化为一个函数
        // 返回值就是转化后的函数，compiledWrapper是一个函数
        const compiledWrapper = vm.runInThisContext(wrapper, {
        filename,
        lineOffset: 0,
        displayErrors: true,
        });
        const dirname = path.dirname(filename);
        // 调用函数，这里一定注意传递进的内容。
        compiledWrapper.call(this.exports, this.exports, this.require, this,
        filename, dirname);
    }
    ```
    这里注意两个地方：
    - 使用 vm 进行模块代码的执行，模块代码外面进行了一层包裹,以便注入一些变量。
    ```javascript
    '(function (exports, require, module, __filename, __dirname) { ',
    '\n});'
    ```
    - 最终执行代码的函数传递的参数
    1. this: compiledWrapper函数是通过 call 调用的，第一个参数就是里面的this，这里我们传入的是 this.exports，也就是 module.exports
    2. exports: compiledWrapper 函数正式接收的第一个参数是 exports，我们传的也是 this.exports,所以 js 文件里面的 exports 也是对module.exports 的一个引用。
    3. require: 这个方法我们传的是 this.require，其实就是KoalaModule.prototype.require 函数，也就是 KoalaModule._load。
    4. module: 我们传入的是 this，也就是当前模块的实例。
    5. __filename：文件所在的绝对路径。
    6. __dirname: 文件所在文件夹的绝对路径。

    以上两点也是我们能在 JS 模块文件里面直接使用这几个变量的原因。
2. 加载 .json
    加载 .json 文件比较简单，直接使用 JSONParse就可以，定位到源码位置 (https://github.com/nodejs/node/blob/master/lib/internal/modules/cjs/loader.js#L1117) ，注意这里不要忘记异常抛出 我们简单实现下：
    ```javascript
    KoalaModule._extensions['.json'] = function (module, filename) {
        const content = fs.readFileSync(filename, 'utf8');
        try {
            module.exports = JSONParse(content);
        } catch (err) {
            throw err;
        }
    }
    ```
3. 加载 .node
    我们自己实现的 C++ 插件或者第三方 C++ 插件，经过编译后会变为.node 扩展名文件。.node 文件的本质是动态链接库，找到对应源码位置(https://github.com/nodejs/node/blob/master/lib/internal/modules/cjs/loader.js#L1135)
    ```javascript
    Module._extensions['.node'] = function(module, filename) {  
        // ...  
        //return process.dlopen(module, path.toNamespacedPath(filename)); 
    };
    ```
    直接调了process.dlopen，该函数在node.js里定义。
    ```javascript
    const rawMethods = internalBinding('process_methods');  
    process.dlopen = rawMethods.dlopen;  
    ```
    找到process_methods模块对应的是node_process_methods.cc。
    ```c++
    env->SetMethod(target, "dlopen", binding::DLOpen);
    ```
    这里也过多分析，放到 Node.js 和 C++ 那些事那篇文章一起讲解。

### 8.返回模块的 module.exports
```javascript
return module.exports;
```
在模块中的代码可以修改 module.exports 的值，不管是从缓存获取，加载原生模块还是普通文件模块，module.exports 都是最终的导出结果

## require 原理理解后的思考
### require加载是同步还是异步
这个问题我直接告诉你同步还是异步，你可能过一阵又忘记了，我之前就是这样。看过源码或者手写一遍就知道了，代码中所有文件相关操作都是使用的同步，举几个例子：
```javascript
fs.existsSync(currentPath)
fs.readFileSync(filename, 'utf8');
```

### exports和module.exports的区别究竟是什么？
在源码中可以发现，
```javascript
compiledWrapper.call(this.exports, this.exports, this.require, this,
    filename, dirname);
```
require在执行文件时候，传递的两个参数 参数一：this.exports = {}; 参数二：this 而this就是 module 所以 module.exports = {};
所以在一个模块没有做任何代码编写之前 exports === module.exports 都是 {} ，共同指向一个引用
看一张图理解这里更清楚：

![](https://api2.mubu.com/v3/document_image/c769a4c7-172e-4c37-aaa1-6c8e21ed684b-3807603.jpg)

但是大多数情况下我们开发时，经常会这样导出
```javascript
exports = {
    name:'kaola'
}
```
或者这样写
```javascript
module.export = {
    value:'程序员成长指北‘
}
```
上面这两种情况就是给 exports 或者 module.exports 重新赋值了，改变了引用地址，两个属性就不再===。
关键点：require一个文件，之前在手写代码时你会发现，最终导出的内容是return module.exports;.所以你对exports的重新赋值不会改变模块的导出内容，只是改变了exports这个变量而已。导出内容永远是module.exports

### require 会不会造成循环引用的问题
自行去看源码实现中的第 7 步，应就懂了。

### require是怎么来的，为什么可以直接在一个文件中使用require
require 到的文件，在 vm 模块最终执行的时，对代码进行了一层包裹，并且把对应的参数传递进去执行。
包裹后的代码：
```javascript
`(function (exports, require, module, __filename, __dirname) { ${script} \n});`
```
执行时的代码：
```javascript
const wrapper = this.wrap(content);    // 获取包装后函数体
// vm是nodejs的虚拟机模块，runInThisContext方法可以接受一个字符串并将它转化为一个函数
const compiledWrapper = vm.runInThisContext(wrapper, {
    filename,
    lineOffset: 0,
    displayErrors: true,
});

const dirname = path.dirname(filename);

compiledWrapper.call(this.exports, this.exports, this.require, this,
    filename, dirname);
```
在这里小伙伴对 vm 模块有疑问的话可以看我的下一篇文章 冷门 Node.js模块 vm 学习必不可少。
通过代码发现 require 函数实际已经传递到了执行的 module 文件中，所以require 在 module 文件中可以直接调用了，同时也应该明白了为什么那几个变量可以直接获取了 dirname,filename 等。

## 补充

| 区别                | require/exports                                              | import/export                                                |
| :------------------ | :----------------------------------------------------------- | :----------------------------------------------------------- |
| 出现的时间/地点不同 | 2009年出生 出自CommonJS                                      | 2015年出生 出自ECMAScript2015(ES6)                           |
| 不同端的使用限制    | 100                                                          | 999                                                          |
| 加载方式            | 同步加载，运行时动态加载，加载的是一个对象，对象需要在脚本运行完成后才会生成 | 异步加载，静态编译，ES6 模块不是对象，它的对外接口只是一种静态定义，在代码静态解析阶段就会生成 |
| 输出对比            | 输出的是一个值的拷贝，一旦输出一个值，模块内部的变化不会影响到这个值 | 输出的是值的引用，JS 引擎对脚本静态分析的时候，遇到模块加载命令import，就会生成一个只读引用。等到脚本真正执行时，再根据这个只读引用，到被加载的那个模块里面去取值。若文件引用的模块值改变，require 引入的模块值不会改变，而 import 引入的模块值会改变。 |
| 使用方式            | 上面手写过程中已经说了使用方式                               | import的使用方式                                             |

## 总结
看源码最好带着一些目的，开篇的一些问题只知道结果并不都知道原因就是我写本文的目的，看源码的过程中也能学到一些内容，比如一些地方你会发现源码是比你写的好，嘿嘿。require 的源码中还是有很多细节点可以学习和分析的，比如这里忽略了 isMain 主文件判断，启动时候 require 的使用(这个会在另一篇文章 Node.js 的启动源码分析中介绍)，以及在 load 方法中执行模块文件时候使用到的 proxy，都可以学习下,本文是一个思路。

## 参考链接
[彻底搞懂 Node.js 中的 Require 机制(源码分析到手写实践)](https://mp.weixin.qq.com/s/NHYfKdxnL59j3l8zJpLV1A)