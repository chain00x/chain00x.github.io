### libmsaoaidsec.so

```
function hook_dlopen() {
  var interceptor = Interceptor.attach(Module.findExportByName(null, "android_dlopen_ext"),
      {
          onEnter: function (args) {
              var pathptr = args[0];
              if (pathptr !== undefined && pathptr != null) {
                  var path = ptr(pathptr).readCString();
                  console.log("[LOAD]", path)
                  if (path.indexOf("libmsaoaidsec.so") > -1) {
                    replace_thread()
                  }
              }
          },
      }
  )
  return interceptor
}

function replace_thread() {
  var pthread_create_addr = Module.findExportByName(null, "pthread_create");
  var pthread_create = new NativeFunction(pthread_create_addr, "int", ["pointer", "pointer", "pointer", "pointer"]);
  Interceptor.replace(pthread_create_addr, new NativeCallback((parg0, parg1, parg2, parg3) => {
          console.log(parg2);
          if(parg2.toString().indexOf('ee4')>-1){
            return 0;
          }
          else{
            return pthread_create(parg0, parg1, parg2, parg3);
          }
      
  }, "int", ["pointer", "pointer", "pointer", "pointer"]));
}

var dlopen_interceptor = hook_dlopen()
```

```
function hook_pthread_create() {
    var pthread_create_addr = Module.findExportByName(null, "pthread_create");
    const __android_log_print_ptr = Module.findExportByName(null, '__android_log_print')
    const cm = new CModule(`
    #include <gum/guminterceptor.h>
    extern void onMessageStr (const gchar * message);
    extern void onMessagePtr (void * message);
    extern void onMessageInt (int a);
    extern int __android_log_print(int prio, const char* tag, const char* fmt, ...);
    void hello(){
    }
    void onEnter (GumInvocationContext * ic)
      {
        // void* arg2,arg3;
        if((((int)gum_invocation_context_get_nth_argument(ic, 2))&0xfff)==0x599){
            onMessageStr("replace success");
            gum_invocation_context_replace_nth_argument(ic,2,(gpointer)hello);
        }
        // arg2 = gum_invocation_context_get_nth_argument (ic, 2);
        onMessagePtr(gum_invocation_context_get_nth_argument (ic, 2));
        // arg0 = (int)gum_invocation_context_get_nth_argument (ic, 2);
        // gum_invocation_context_replace_nth_argument(ic,2,(gpointer)100);
        // arg1 = (int)gum_invocation_context_get_nth_argument (ic, 3);
        // log ("function add arg0=%d arg1=%d ", arg0,arg1);
        // __android_log_print(3,"isDebug","arg0=%d,arg1=%d",arg0,arg1);
      }
      void
      onLeave (GumInvocationContext * ic)
      {
        int result;
        result = (int) gum_invocation_context_get_return_value (ic);
        onMessageInt(result);
        // gum_invocation_context_replace_return_value (ic,(gpointer)100);
      }`, {
        __android_log_print: __android_log_print_ptr,
        onMessageStr: new NativeCallback(messagePtr => {
            const message = messagePtr.readUtf8String();
            // console.log('onMessageStr:', message);
        }, 'void', ['pointer']),
        onMessagePtr: new NativeCallback(messagePtr => {
            // console.log('onMessagePtr:', messagePtr ,hexdump(messagePtr));
        }, 'void', ['pointer']),
        onMessageInt: new NativeCallback(messageInt => {
            // console.log('onMessageInt:', messageInt);
        }, 'void', ['int']),
    });
    Interceptor.attach(pthread_create_addr, cm);
}

function replace_thread() {
    var pthread_create_addr = Module.findExportByName(null, "pthread_create");
    var pthread_create = new NativeFunction(pthread_create_addr, "int", ["pointer", "pointer", "pointer", "pointer"]);
    Interceptor.replace(pthread_create_addr, new NativeCallback((parg0, parg1, parg2, parg3) => {
        try{
            var so_name = Process.findModuleByAddress(parg2).name;
            var so_base = Module.getBaseAddress(so_name);
            var offset = (parg2 - so_base);
            var PC = 0;
            console.log("normal find thread func offset", so_name, parg2,offset, offset.toString(16));
            // i加密
            if(
            (so_name.indexOf("libmsaoaidsec.so")>-1 && offset===110308)
            ||(so_name.indexOf("libmsaoaidsec.so")>-1 && offset===107892)
            ){
            }
            else{
                PC = pthread_create(parg0, parg1, parg2, parg3);
            }
            } catch(err) {
                PC = pthread_create(parg0, parg1, parg2, parg3);
            }
        
        return PC;
    }, "int", ["pointer", "pointer", "pointer", "pointer"]));
}


setImmediate(replace_thread)
```

### libexecmain.so、libexec.so

https://iiong.com/reinforce-android-applications-for-unpacking-learning/

```
var soaddr = null;
function hook_dlopen(soName = "") {
  Interceptor.attach(Module.findExportByName(null, "android_dlopen_ext"), {
    onEnter: function (args) {
      var pathptr = args[0];

      if (pathptr !== undefined && pathptr != null) {
        var path = ptr(pathptr).readCString();
        if (path.indexOf(soName) != -1) {
          this.hook = true;
        }
        console.log(path);
      }
    },
    onLeave: function (ret) {
      if ((this.hook = true)) {
        soaddr = Module.findBaseAddress(soName);
        hook_pthread_create(soName);
      }
    },
  });
}

function hook_pthread_create(soName) {
  Interceptor.attach(Module.findExportByName("libc.so", "pthread_create"), {
    onEnter(args) {
      var func_addr = args[2];

      
      // console.log("The thread function address is " + offes);
      try{
        var offes = func_addr.sub(soaddr);
        if (offes == 0x329cc || offes == 0x454ac) {
            Interceptor.replace(
              func_addr,
              new NativeCallback(
                function () {
                  console.log("in replaces");
                  setTimeout(function () {
                    console.log("脱壳中。。。");
                  }, 1000 * 10);
                },
                "void",
                []
              )
            );
          }

      }
      catch{

      }
      
    },
  });
}
setImmediate(hook_dlopen, "libexec.so");
```

### libdexjni.so

https://showfaker.top/2024/03/20/bb-reverse-bypass/

```
function printStack() {
    Java.perform(function () {
      var Exception = Java.use("java.lang.Exception");
      var ins = Exception.$new("Exception");
      var straces = ins.getStackTrace();
      if (straces != undefined && straces != null) {
        var strace = straces.toString();
        var replaceStr = strace.replace(/,/g, "\r\n");
        console.log(
          "============================= Stack start ======================="
        );
        console.log(replaceStr);
        console.log(
          "============================= Stack end =======================\r\n"
        );
        Exception.$dispose();
      }
    });
  }
  function hook_dlopen() {
    var module = Module.findExportByName(null, "android_dlopen_ext");
    Interceptor.attach(module,
      {
        onEnter: function (args) {
          var pathptr = args[0];
          // console.log(ptr(pathptr).readCString());
          if (ptr(pathptr).readCString().indexOf("libmsaoaidsec.so") != -1) {
  
            // print_thread()
            locate_init()
            console.log("android_dlopen_ext:", ptr(pathptr).readCString());
  
          }
  
          // if (pathptr !== undefined && pathptr != null) {
          //     var path = ptr(pathptr).readCString();
          //     console.log("load " + path);
          // }
        },
        onLeave: function (args) {
          // var base_addr = Module.findBaseAddress('libmsaoaidsec.so');
          // console.log("onLeave  base_addr:  ",base_addr);  
        }
      }
    );
  }
  var moduleMap = {}
  var dexjni_start = false
  function hook_constructor() {
  
    if (Process.pointerSize == 4) {
      var linker = Process.findModuleByName("linker");
    } else {
      var linker = Process.findModuleByName("linker64");
    }
  
    var addr_call_function = null;
    var addr_g_ld_debug_verbosity = null;
    var addr_async_safe_format_log = null;
    var linker_log = null;
    console.log("linker:", linker.base);
    if (linker) {
      var symbols = linker.enumerateSymbols();
      for (var i = 0; i < symbols.length; i++) {
        var name = symbols[i].name;
        // console.log(name)
        if (name.indexOf("call_function") >= 0) {
          addr_call_function = symbols[i].address;
        } else if (name.indexOf("g_ld_debug_verbosity") >= 0) {
          addr_g_ld_debug_verbosity = symbols[i].address;
  
          ptr(addr_g_ld_debug_verbosity).writeInt(2);
        } else if (
          name.indexOf("async_safe_format_log") >= 0 &&
          name.indexOf("va_list") >= 0
        ) {
          addr_async_safe_format_log = symbols[i].address;
          console.log("addr_async_safe_format_log", addr_async_safe_format_log);
        } else if (
          name.indexOf("linker") >= 0 &&
          name.indexOf("log") >= 0 &&
          name.indexOf("va_list") < 0 &&
          name.indexOf("cpp") < 0 &&
          name.indexOf("logger") < 0
        ) {
          console.log("name:", name);
          linker_log = symbols[i].address;
        }
      }
    }
  
    if (linker_log) {
      Interceptor.attach(linker_log, {
        onEnter: function (args) {
          this.log_level = args[0];
          this.fmt = ptr(args[1]).readCString();
          this.tag = ptr(args[2]).readCString();
  
  
          // console.log("this.tag",this.tag)
          if (
            this.fmt.indexOf("Calling") >= 0 &&
            this.fmt.indexOf("for") >= 0 &&
            this.tag.indexOf("DT_INIT_ARRAY") >= 0
          ) {
            // console.log("c-tor")
            this.size = args[3]
            this.path = ptr(args[5]).readCString()
            console.log("this.tag", this.tag);
            console.log("this.fmt", this.fmt);
            console.log("size:", args[3]);
            // console.log("base:",args[4]);
            console.log("this.path", ptr(args[5]).readCString());
  
            if (this.path.endsWith(".so")) {
              var moduleName = this.path.split('/').pop()
              console.log("module Name:", moduleName)
              var myModule = Process.findModuleByName(moduleName);
              moduleMap[moduleName] = [myModule.base, myModule.size]
            }
            // (this.array_addr = ptr(args[4])), // func_type
            //     console.log("array addr:", this.array_addr);
            if (ptr(args[5]).readCString().indexOf("libDexHelper.so") != -1) {
              // hook_thread()
              var libDexHelperaddr = Process.findModuleByName("libDexHelper.so");
              console.log("libDexHelperaddr:", libDexHelperaddr.base);
              // hook_thread();
              // hook_anti_frida_replace(libDexHelperaddr.base.add(0x46dcc));
              // hook46DCC(libDexHelperaddr.base.add(0x46DCC))
              hook_46DCC_body_replace(libDexHelperaddr.base.add(0x46DCC))
              hook3A640(libDexHelperaddr.base.add(0x3A640))
              hook_44fb0_body_replace(libDexHelperaddr.base.add(0x44fb0))
              // hook_anti_fork_replace(libDexHelperaddr.base.add(0x2cec0));
              hook_4EC60_body_replace(libDexHelperaddr.base.add(0x4ec60))
              patch_fork(libDexHelperaddr.base.add(0x3d0ac));
              patch_fork(libDexHelperaddr.base.add(0x3d2b0));
              hook_anti_waitbody_replace(libDexHelperaddr.base.add(0x4f04c));
              hook_anti_thread_body_replace(libDexHelperaddr.base.add(0x32d08));
              hook_4F268_body_replace(libDexHelperaddr.base.add(0x4f268));
              hook_anti_waitkill_replace(libDexHelperaddr.base.add(0x4eef8));
              patch_read(libDexHelperaddr.base.add(0x3d0ec));
  
            }
            if (ptr(args[5]).readCString().indexOf("libdexjni.so") != -1) {
              var libdexjni = Process.findModuleByName("libdexjni.so");
              console.log("libdexjni addr:", libdexjni.base);
              // Interceptor.detachAll()
              dexjni_start = true;
            }
  
            if (ptr(args[5]).readCString().indexOf("libzwmonitor.so") != -1) {
              printStack()
              print_thread()
            }
          }
        },
        onLeave: function (retval) { },
      });
    }
  }
  // hook_thread()
//   hook_system_property_get();
  setImmediate(hook_constructor);
  // hook_dlopen()
  
  function hook3A640(funaddr) {
    Interceptor.attach(funaddr, {
      onEnter: function (args) {
  
      },
      onLeave: function (retval) {
        console.log("3A640 end")
        // hook_thread()
        // hook_dlopen()
        // hook_msao()
      },
    });
  }
  
  this.flag = false;
  function hook46DCC(funaddr) {
    console.log("Hook 46DCC")
    Interceptor.attach(funaddr, {
      onEnter: function (args) {
        if (args[2] != 0) {
          console.log("46DCC:", Memory.readCString(args[2]));
          if (args[2].readCString().indexOf("pthread_create") != -1) {
            this.flag = true;
          }
        }
      },
      onLeave: function (retval) {
        if (this.flag) {
          console.log("ret is replaced");
          retval.replace(0);
        }
      },
    });
  }
  
  function hook_46DCC_body_replace(addr) {
    console.log("replace 46DCC_addr :", addr);
    Interceptor.replace(
      addr,
      new NativeCallback(
        function (a1) {
          console.log("replace 46DCC_body success");
          return 0;
        },
        "int",
        ["int", "pointer", "pointer"]
      )
    );
  }
  
  function hook_4EC60_body_replace(addr) {
    console.log("replace anti_addr :", addr);
    Interceptor.replace(
      addr,
      new NativeCallback(
        function (a1) {
          console.log("replace 4EC60_body success");
          return;
        },
        "void",
        ["pointer"]
      )
    );
  }
  
  function hook_44fb0_body_replace(addr) {
    console.log("replace 0x44fb0 :", addr);
    Interceptor.replace(
      addr,
      new NativeCallback(
        function (a1) {
          console.log("replace 0x44fb0 success");
          return 0;
        },
        "int",
        ["pointer"]
      )
    );
  }
  
  function hook_anti_frida_replace(addr) {
    console.log("start antifrida :", addr);
    Interceptor.replace(
      addr,
      new NativeCallback(
        function (a1) {
          console.log("replace antifrida success");
          return;
        },
        "void",
        ["pointer"]
      )
    );
  }
  
  function hook_anti_fork_replace(addr) {
    //0x2cec0
    console.log("replace anti_fork addr :", addr);
    Interceptor.replace(
      addr,
      new NativeCallback(
        function () {
          console.log("replace fork success");
          return 123; //父进程返回虚假子进程pid，且不创建子进程
        },
        "int",
        []
      )
    );
  }
  function hook_anti_waitbody_replace(addr) {
    //0x4f04c
    console.log("replace anti_waitbody :", addr);
    Interceptor.replace(
      addr,
      new NativeCallback(
        function () {
          console.log("replace waitbody success");
          return 123; //父进程返回虚假子进程pid，且不创建子进程
        },
        "void",
        []
      )
    );
  }
  function hook_anti_waitkill_replace(addr) {
    //0x4eef8
    console.log("replace anti_addr :", addr);
    Interceptor.replace(
      addr,
      new NativeCallback(
        function () {
          console.log("replace waitbody success");
          return 123; //父进程返回虚假子进程pid，且不创建子进程
        },
        "void",
        []
      )
    );
  }
  function hook_anti_thread_body_replace(addr) {
    //anti_thread_body 0x32d08
    console.log("replace anti_addr :", addr);
    Interceptor.replace(
      addr,
      new NativeCallback(
        function (a1) {
          console.log("replace antifrida success");
          return 0;
        },
        "int",
        ["pointer"]
      )
    );
  }
  
  function hook_4F268_body_replace(addr) {
    //anti_thread_body 0x32d08
    console.log("replace anti_addr :", addr);
    Interceptor.replace(
      addr,
      new NativeCallback(
        function (a1) {
          console.log("replace antifrida success");
          return 0;
        },
        "void",
        []
      )
    );
  }
  
  function patch_read(addr) {
    // var addr = 0x3d0ec
    Memory.patchCode(addr, 4, function (code) {
      // var cw = new X86Writer(code, { pc: getLivesLeft });
      var cw = new Arm64Writer(code, { pc: addr });
      // cw.putMovRegU32("eax", 9000);
      // cw.putRet();
      // cw.putInstruction(0x00008052);
      cw.putInstruction(0x52800000);
      cw.flush();
    });
  }
  
  function patch_fork(addr) {
    // var addr = 0x3d0ec
    Memory.patchCode(addr, 4, function (code) {
      // var cw = new X86Writer(code, { pc: getLivesLeft });
      var cw = new Arm64Writer(code, { pc: addr });
      // cw.putMovRegU32("eax", 9000);
      // cw.putRet();
      // cw.putInstruction(0x600F8052);
      cw.putInstruction(0x52800f60);
      cw.flush();
    });
  }
  function hook_system_property_get() {
    var system_property_get = Module.findExportByName(
      "libc.so",
      "__system_property_get"
    );
    Interceptor.attach(system_property_get, {
      onEnter: function (args) {
        this.flag_debug = false;
        this.property = args[1]
        console.log("system_property_get:", Memory.readCString(args[0]));
        if (Memory.readCString(args[0]).indexOf("ro.debuggable") != -1) {
          this.flag_debug = true;
        }
        if (Memory.readCString(args[0]).indexOf("ro.build.version.release_or_codename") != -1) {
          console.log("------------------------------")
          // dump("libmsaoaidsec.so")
  
          var secmodule = Process.findModuleByName("libmsaoaidsec.so")
          console.log("secmodule:", secmodule.base)
          hook_thread(secmodule.base)
        }
      },
      onLeave: function (retval) {
        if (this.flag_debug) {
          retval.replace(0);
        }
        console.log("property:", Memory.readCString(this.property))
        console.log("ret:", retval);
      },
    });
  
  }
  
  function locate_init() {
    let secmodule = null
    Interceptor.attach(Module.findExportByName(null, "strstr"),
      {
        // _system_property_get("ro.build.version.sdk", v1);
        onEnter: function (args) {
          console.log("strstr is called!")
          secmodule = Process.findModuleByName("libmsaoaidsec.so")
          var name = args[1];
          console.log("strstr args[0]:", ptr(args[0]).readCString())
          console.log("strstr args[1]:", ptr(args[1]).readCString())
          if (name !== undefined && name != null) {
            name = ptr(name).readCString();
            if (name.indexOf("2022-02") >= 0) {
              console.log("2022-02 is reached!")
              console.log("base:  ", secmodule.base, "  name: ", secmodule.name);
              // hook_thread(secmodule.base)
  
  
            }
          }
  
        },
        onLeave: function () {
  
        }
      }
    );
  }
  function dump(name) {
    var dir = "/data/data/com.hanweb.android.zhejiang.activity/files"
    var libxx = Process.getModuleByName(name);
    console.log("*****************************************************");
    console.log("name: " + libxx.name);
    console.log("base: " + libxx.base);
    console.log("size: " + ptr(libxx.size));
  
    var file_path = dir + "/" + libxx.name + "_" + libxx.base + "_" + ptr(libxx.size) + ".so";
    console.log(file_path);
  
    var file_handle = new File(file_path, "wb");
    if (file_handle && file_handle != null) {
      Memory.protect(ptr(libxx.base), libxx.size, 'rwx');
      var libso_buffer = ptr(libxx.base).readByteArray(libxx.size);
      file_handle.write(libso_buffer);
      file_handle.flush();
      file_handle.close();
      console.log("[dump]:", file_path);
    }
  }
  
  
  function isInModule(address) {
    for (let i in moduleMap) {
      // console.log("add res:",parseInt(moduleMap[i][0]) + parseInt(moduleMap[i][1]))
      if (address >= parseInt(moduleMap[i][0]) && address <= parseInt(moduleMap[i][0]) + parseInt(moduleMap[i][1])) {
        // console.log("address ",address,"is in ",i)
        return true
      }
    }
    // console.log("address ",address,"is not in any module")
    return false
  }
  var hook_thread_flag = true
  function hook_thread(base_addr) {
    var pt_create_func = Module.findExportByName(null, 'pthread_create');
    var detect_frida_loop_addr = null;
    // console.log('pt_create_func:',pt_create_func);
  
    if (hook_thread_flag) {
      Interceptor.attach(pt_create_func, {
        onEnter: function () {
          console.log("hook_thread:", this.context.x2)
  
          // isInModule(this.context.x2)
          if (dexjni_start && isInModule(this.context.x2)) {
            console.log(Thread.backtrace(this.context, Backtracer.ACCURATE).map(DebugSymbol.fromAddress).join('\n') + '\n');
          }
          if (this.context.x2.compare(base_addr.add(0x175F8)) == 0 || this.context.x2.compare(base_addr.add(0x16D30)) == 0 || this.context.x2.compare(base_addr.add(0x163DC)) == 0) {
            hook_anti_msao_replace(this.context.x2);
          }
  
          // hook_anti_msao_replace(this.context.x2);
        },
        onLeave: function (retval) {
          // console.log('retval',retval);
        }
      })
    }
    hook_thread_flag = false;
  }
  
  function hook_anti_msao_replace(addr) {
    console.log('replace anti_msao_addr :', addr);
    Interceptor.replace(addr, new NativeCallback(function (a1) {
      console.log('anti_msao_addr replace success');
      return;
    }, 'void', []));
  
  }
  
  function print_thread() {
  
    // var pt_create_func = Module.findExportByName("libc.so", 'strcpy')
    var pt_create_func = Module.findExportByName(null, 'pthread_create');
    console.log("pt_create_func 2:", pt_create_func)
  
    Interceptor.attach(pt_create_func, {
      onEnter: function (args) {
        console.log("thread address", args[2])
        // console.log(Thread.backtrace(this.context, Backtracer.ACCURATE).map(DebugSymbol.fromAddress).join('\n') + '\n');
        // console.log("-----------------------------------------------")
  
      },
      onLeave: function (retval) {
  
      }
    })
  }
```

### libDexHelper.so

```
setTimeout(function() {
    var freadAddr = Module.findExportByName('libc.so', "fread");
    
    Interceptor.attach(freadAddr, {
        onEnter: function(args) {
            if (Thread.backtrace(this.context, Backtracer.ACCURATE).map(DebugSymbol.fromAddress).indexOf('libDexHelper.so')>-1) {
                const newData = new Uint8Array(this.totalSize).fill(0x00);
                Memory.writeByteArray(this.buffer, newData);
            }
        }
    });
}, 1000);
```
