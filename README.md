# Android-Anti-AntiTrace
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
 
    do_native_hooks_libfoo();
 
    send("ret: " + ret);
 
}, "int", ["pointer", "pointer", "pointer", "pointer"]));
```
