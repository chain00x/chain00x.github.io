原生
```
console.log("脚本加载成功");
function main(){
    Java.perform(function(){
        var WebView = Java.use('android.webkit.WebView');
        WebView.$init.overload('android.content.Context').implementation = function(a){
            console.log('===hook===')
            var result = this.$init(a);
            this.setWebContentsDebuggingEnabled(true);
            return result;
        }
        WebView.$init.overload('android.content.Context', 'android.util.AttributeSet').implementation = function(a,b){
            console.log('===hook===')
            var result = this.$init(a,b);
            this.setWebContentsDebuggingEnabled(true);
            return result;
        }
        WebView.$init.overload('android.content.Context', 'android.util.AttributeSet', 'int').implementation = function(a,b,c){
            console.log('===hook===')
            var result = this.$init(a,b,c);
            this.setWebContentsDebuggingEnabled(true);
            return result;
        }
        WebView.$init.overload('android.content.Context', 'android.util.AttributeSet', 'int', 'int').implementation = function(a,b,c,d){
            console.log('===hook===')
            var result = this.$init(a,b,c,d);
            this.setWebContentsDebuggingEnabled(true);
            return result;
        }
        WebView.$init.overload('android.content.Context', 'android.util.AttributeSet', 'int', 'boolean').implementation = function(a,b,c,d){
            console.log('===hook===')
            var result = this.$init(a,b,c,d);
            this.setWebContentsDebuggingEnabled(true);
            return result;
        }
        WebView.$init.overload('android.content.Context', 'android.util.AttributeSet', 'int', 'java.util.Map', 'boolean').implementation = function(a,b,c,d,e){
            console.log('===hook===')
            var result = this.$init(a,b,c,d,e);
            this.setWebContentsDebuggingEnabled(true);
            return result;
        }
        WebView.$init.overload('android.content.Context', 'android.util.AttributeSet', 'int', 'int', 'java.util.Map', 'boolean').implementation = function(a,b,c,d,e,f){
            console.log('===hook===')
            var result = this.$init(a,b,c,d,e,f);
            this.setWebContentsDebuggingEnabled(true);
            return result;
        }
        WebView["setWebContentsDebuggingEnabled"].implementation = function (z) {
            console.log('===hook===')
            this["setWebContentsDebuggingEnabled"](true);
        };
    });
}
setImmediate(main);
```
腾讯x5
```
console.log("脚本加载成功");
function main(){
    Java.perform(function(){
        var WebView = Java.use('com.tencent.smtt.sdk.WebView');
        WebView.$init.overload('android.content.Context').implementation = function(a){
            console.log('===hook===')
            var result = this.$init(a);
            this.setWebContentsDebuggingEnabled(true);
            return result;
        }
        WebView.$init.overload('android.content.Context', 'android.util.AttributeSet').implementation = function(a,b){
            console.log('===hook===')
            var result = this.$init(a,b);
            this.setWebContentsDebuggingEnabled(true);
            return result;
        }
        WebView.$init.overload('android.content.Context', 'boolean').implementation = function(a,b){
            console.log('===hook===')
            var result = this.$init(a,b);
            this.setWebContentsDebuggingEnabled(true);
            return result;
        }
        WebView.$init.overload('android.content.Context', 'android.util.AttributeSet', 'int').implementation = function(a,b,c){
            console.log('===hook===')
            var result = this.$init(a,b,c);
            this.setWebContentsDebuggingEnabled(true);
            return result;
        }
        WebView.$init.overload('android.content.Context', 'android.util.AttributeSet', 'int', 'boolean').implementation = function(a,b,c,d){
            console.log('===hook===')
            var result = this.$init(a,b,c,d);
            this.setWebContentsDebuggingEnabled(true);
            return result;
        }
        WebView.$init.overload('android.content.Context', 'android.util.AttributeSet', 'int', 'java.util.Map', 'boolean').implementation = function(a,b,c,d,e){
            console.log('===hook===')
            var result = this.$init(a,b,c,d,e);
            this.setWebContentsDebuggingEnabled(true);
            return result;
        }
        WebView["setWebContentsDebuggingEnabled"].implementation = function (z) {
            console.log('===hook===')
            this["setWebContentsDebuggingEnabled"](true);
        };
    });
}
setImmediate(main);
```
uc
```

console.log("脚本加载成功");
function main(){
    Java.perform(function(){
        var WebView = Java.use('com.uc.webview.export.WebView');
        WebView.$init.overload('android.content.Context').implementation = function(a){
            console.log('===hook===')
            var result = this.$init(a);
            this.setWebContentsDebuggingEnabled(true);
            return result;
        }
        WebView.$init.overload('android.content.Context', 'android.util.AttributeSet').implementation = function(a,b){
            console.log('===hook===')
            var result = this.$init(a,b);
            this.setWebContentsDebuggingEnabled(true);
            return result;
        }
        WebView.$init.overload('android.content.Context', 'boolean').implementation = function(a,b){
            console.log('===hook===')
            var result = this.$init(a,b);
            this.setWebContentsDebuggingEnabled(true);
            return result;
        }
        WebView.$init.overload('android.content.Context', 'android.util.AttributeSet', 'int').implementation = function(a,b,c){
            console.log('===hook===')
            var result = this.$init(a,b,c);
            this.setWebContentsDebuggingEnabled(true);
            return result;
        }
        WebView.$init.overload('android.content.Context', 'android.util.AttributeSet', 'boolean').implementation = function(a,b,c){
            console.log('===hook===')
            var result = this.$init(a,b,c);
            this.setWebContentsDebuggingEnabled(true);
            return result;
        }
        WebView.$init.overload('android.content.Context', 'android.util.AttributeSet', 'int', 'boolean').implementation = function(a,b,c,d){
            console.log('===hook===')
            var result = this.$init(a,b,c,d);
            this.setWebContentsDebuggingEnabled(true);
            return result;
        }
        WebView.$init.overload('android.content.Context', 'android.util.AttributeSet', 'boolean', 'int').implementation = function(a,b,c,d){
            console.log('===hook===')
            var result = this.$init(a,b,c,d);
            this.setWebContentsDebuggingEnabled(true);
            return result;
        }
        WebView.$init.overload('android.content.Context', 'android.util.AttributeSet', 'int', 'boolean', 'byte').implementation = function(a,b,c,d,e){
            console.log('===hook===')
            var result = this.$init(a,b,c,d,e);
            this.setWebContentsDebuggingEnabled(true);
            return result;
        }
        
        WebView["setWebContentsDebuggingEnabled"].implementation = function (z) {
            console.log('===hook===')
            this["setWebContentsDebuggingEnabled"](true);
        };
    });
}
setImmediate(main);
```
