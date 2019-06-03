# Android-Anti-AntiTrace
```javascript
// int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start_routine) (void *), void *arg);
var p_pthread_create = Module.findExportByName("libc.so","pthread_create");
Interceptor.attach(ptr(p_pthread_create), {
    onEnter: function (args) {
        this.thread        = args[0];
        this.attr          = args[1];
        this.start_routine = args[2];
        this.arg           = args[3];
        this.fakeRet       = Boolean(0);
        send("onEnter() pthread_create(" + this.thread.toString() + ", " + this.attr.toString() + ", "
            + this.start_routine.toString() + ", " + this.arg.toString() + ");");
 
        if (parseInt(this.attr) == 0 && parseInt(this.arg) == 0)
            this.fakeRet = Boolean(1);
 
    },
    onLeave: function (retval) {
        send(retval);
        send("onLeave() pthread_create");
        if (this.fakeRet == 1) {
            var fakeRet = ptr(0);
            send("pthread_create real ret: " + retval);
            send("pthread_create fake ret: " + fakeRet);
            return fakeRet;
        }
        return retval;
    }
});
```
```javascript
// int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start_routine) (void *), void *arg);
var p_pthread_create = Module.findExportByName("libc.so", "pthread_create");
var pthread_create = new NativeFunction( p_pthread_create, "int", ["pointer", "pointer", "pointer", "pointer"]);
send("NativeFunction pthread_create() replaced @ " + pthread_create);
 
Interceptor.replace( p_pthread_create, new NativeCallback(function (ptr0, ptr1, ptr2, ptr3) {
    send("pthread_create() overloaded");
    var ret = ptr(0);
    if (ptr1.isNull() && ptr3.isNull()) {
        send("loading fake pthread_create because ptr1 and ptr3 are equal to 0!");
    } else {
        send("loading real pthread_create()");
        ret = pthread_create(ptr0,ptr1,ptr2,ptr3);
    }
 
 
    send("ret: " + ret);
 
}, "int", ["pointer", "pointer", "pointer", "pointer"]));
```
```javascript
//long ptrace(enum __ptrace_request request, pid_t pid,void *addr, void *data);
var p_ptrace = Module.findExportByName("libc.so", "ptrace");
var ptrace = new NativeFunction( p_pthread_create, "int", ["int", "int", "pointer", "pointer"]);
send("NativeFunction ptrace() replaced @ " + ptrace);
 
Interceptor.replace( p_ptrace, new NativeCallback(function (ptr0, ptr1, ptr2, ptr3) {
    send("p_ptrace() overloaded");
    var ret = ptr(0);
    if (ptr1.isNull() && ptr3.isNull()) {
        send("loading fake ptrace because ptr1 and ptr3 are equal to 0!");
    } else {
        send("loading real ptrace()");
        ret = ptrace(ptr0,ptr1,ptr2,ptr3);
    }
 
 
    send("ret: " + ret);
 
}, "int", ["int", "int", "pointer", "pointer"]));
```

```javescript
var fgetsPtr = Module.findExportByName("libc.so", "fgets");
var fgets = new NativeFunction(fgetsPtr, 'pointer', ['pointer', 'int', 'pointer']);
Interceptor.replace(fgetsPtr, new NativeCallback(function (buffer, size, fp) {
    var retval = fgets(buffer, size, fp);
    var bufstr = Memory.readUtf8String(buffer);
    if (bufstr.indexOf("TracerPid:") > -1) {
        Memory.writeUtf8String(buffer, "TracerPid:\t0");
        // dmLogout("tracerpid replaced: " + Memory.readUtf8String(buffer));
    }
    return retval;
}, 'pointer', ['pointer', 'int', 'pointer']));
```

