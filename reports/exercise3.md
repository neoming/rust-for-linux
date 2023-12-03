# 在`rust for linux`中使用自定义`bindings`

## 1. 相关修改说明
![](img/add_self_bindings.png)

**bindings实现相关改动**
```
include/linux/neoming_bindgen.h     # 自定义bindings实现
rust/bindings/bindings_helper.h     # 将自定义bindings注册
rust/helpers.c                      # EXPORT 自定义bindings
```
```c
static int neoming_bindgen_test(void)
{
	printk("Hello from neoming's bindgen\n");
	return 0;
}
```

**使用自定义`module`调用自定义`bindings` 相关改动**
```
samples/rust/Kconfig                # 注册module到menuconfig
samples/rust/Makefile               # 添加rust_helloworld.rs到makefile
samples/rust/rust_helloworld.rs     # 使用自定义bindings进行打印
```

```rust
impl kernel::Module for RustHelloWorld {
    fn init(_name: &'static CStr, _module: &'static ThisModule) -> Result<Self> {
        pr_info!("Rust hello world macros sample (init)\n");
        unsafe {
            bindings::neoming_bindgen_test();
        }
        Ok(RustHelloWorld)
    }
}
```

## 2.运行结果
```
(kernel) >insmod rust_helloworld.ko
[   89.254266] rust_hello_world: Rust hello world macros sample (init)
[   89.254443] Hello from neoming's bindgen
(kernel) >rmmod rust_helloworld.ko
[   92.275270] rust_hello_world: Rust hello world macros sample (exit)
(kernel) >
```