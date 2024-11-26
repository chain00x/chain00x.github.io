### okhttp拦截器

```js
let RequestBody = Java.use("okhttp3.RequestBody");
RequestBody["create"].overload('okhttp3.MediaType', 'java.lang.String').implementation = function (mediaType, str) {
    if (str.indexOf('{"cr":') == -1) {
        console.log(`${str}\n`);
    }
    return this["create"](mediaType, str);
};
//get body
let FlowBaffleInterceptor = Java.use("com.xwbank.ebank.http.interceptor.FlowBaffleInterceptor");
FlowBaffleInterceptor["intercept"].implementation = function (chain) {
    let ResponseBody = Java.use("okhttp3.ResponseBody");
    let buffer = Java.use("okio.Buffer").$new();
    let requestBody = chain.request().body();
    requestBody.writeTo(buffer);
    let originalBody = buffer.readUtf8();
    console.log(`\nrequest:${chain.request().method()} \n${chain.request().url().toString()}\n${originalBody}`);
//modify body
    let result;
    if (chain.request().url().toString().indexOf('allSupportFainancialProducts') > -1) {
        let MediaType = Java.use("okhttp3.MediaType");
        let newMediaType = MediaType.parse("application/json; charset=utf-8");
        let testString = '{"productCodes":["F021009001001001","F022001001002001","F022001006023001","F022001003003001",F022001006013001]}';
        let newRequestBody = RequestBody.create(newMediaType, testString);

        // 获取 Builder
        let Builder = Java.use("okhttp3.Request$Builder").$new();
        let newRequest = Builder
            .url(chain.request().url())
            .method(chain.request().method(), newRequestBody)
            .build();
        result = chain.proceed(newRequest); // 使用新请求
    } else {
        result = chain.proceed(chain.request()); // 使用原请求
    }
//get response body
    let responseBody = result.body();
    let source = responseBody.source();
    source.request(0x7fffffffffffffff);
    let buffer2 = source.getBuffer();
    let responseBodyString = buffer2.readUtf8();
    console.log(`\nresponse:\n${responseBodyString}`);

    let newResponseBody = ResponseBody.create(responseBody.contentType(), responseBodyString);
    return result.newBuilder()
        .body(newResponseBody)
        .build();
};
```

### 动态加载dex

```js
function hook() {
    Java.perform(function () {
        Java.enumerateClassLoadersSync().forEach(function (classloader) {
            try {
                console.log("classloader", classloader);
                classloader.loadClass("com.kanxue.encrypt01.MainActivity");
                Java.classFactory.loader = classloader;
                var mainActivityClass = Java.use("com.kanxue.encrypt01.MainActivity");
                console.log("mainActivityClass", mainActivityClass);
            } catch (error) {
                console.log("error", error);
            }
        });
    })
}
```

### hook文件操作
```js
Java.perform(function () {
    // 加入延迟，确保库已加载
    setTimeout(function() {
        var openFiles = {}; // 用于存储打开文件的映射

        // hook open 函数
        var openAddr = Module.findExportByName(null, "open");
        Interceptor.attach(openAddr, {
            onEnter: function (args) {
                var caller = this.context.lr;
                var callerModule = Process.getModuleByAddress(caller);

                // 仅在来自 libDexHelper.so 的调用时输出
                if (callerModule && callerModule.name === "libDexHelper.so") {
                    this.path = args[0].readUtf8String(); // 获取文件路径
                    var fd = args[1].toInt32();
                    openFiles[fd] = this.path; // 保存文件描述符与路径的映射
                    console.log("[open] Called from libDexHelper.so, path: " + this.path);
                }
            }
        });

        // hook read 函数
        var readAddr = Module.findExportByName(null, "read");
        Interceptor.attach(readAddr, {
            onEnter: function (args) {
                var caller = this.context.lr;
                var callerModule = Process.getModuleByAddress(caller);

                // 仅在来自 libDexHelper.so 的调用时输出
                if (callerModule && callerModule.name === "libDexHelper.so") {
                    console.log("[read] Called from libDexHelper.so, fd: " + args[0].toInt32());
                }
            }
        });

        // hook close 函数
        var closeAddr = Module.findExportByName(null, "close");
        Interceptor.attach(closeAddr, {
            onEnter: function (args) {
                var caller = this.context.lr;
                var callerModule = Process.getModuleByAddress(caller);

                // 仅在来自 libDexHelper.so 的调用时输出
                if (callerModule && callerModule.name === "libDexHelper.so") {
                    var fd = args[0].toInt32();
                    var filePath = openFiles[fd]; // 从映射中获取文件路径
                    console.log("[close] Called from libDexHelper.so, fd: " + fd + ", path: " + filePath);
                    delete openFiles[fd]; // 清理映射
                }
            }
        });

        // hook fopen 函数
        var fopenAddr = Module.findExportByName(null, "fopen");
        Interceptor.attach(fopenAddr, {
            onEnter: function (args) {
                var path = args[0].readUtf8String(); // 文件路径
                console.log("[fopen] Called, path: " + path);
            }
        });

        // hook fread 函数
        var freadAddr = Module.findExportByName(null, "fread");
        Interceptor.attach(freadAddr, {
            onEnter: function (args) {
                console.log("[fread] Called, size: " + args[1].toInt32() + ", count: " + args[2].toInt32());
            }
        });

        // hook fclose 函数
        var fcloseAddr = Module.findExportByName(null, "fclose");
        Interceptor.attach(fcloseAddr, {
            onEnter: function (args) {
                console.log("[fclose] Called");
            }
        });

        // hook pread 函数
        var preadAddr = Module.findExportByName(null, "pread");
        Interceptor.attach(preadAddr, {
            onEnter: function (args) {
                console.log("[pread] Called, fd: " + args[0].toInt32());
            }
        });

        console.log("Hook 已经设置");
    }, 1000); // 延迟 1 秒
});
```

### frida hook 所有重载

```js
Java.perform(function () {
    var MyClass = Java.use('skahr.d0');
var overloads  = MyClass["a"].overloads;
for (var i=0;i< overloads.length;i++){
    overloads[i].implementation = function () {
       var params = "";
       for(var j=0;j<arguments.length;j++){
        params = params +arguments[j]+" "
       }
       console.log("params",params)
        
        return this.a.apply(this,arguments);
    };
}
});
```
