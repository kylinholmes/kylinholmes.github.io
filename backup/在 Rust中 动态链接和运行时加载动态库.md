## 动态链接crate

**假设动态库是 impls**
**需要链接到 impls的crate是abi**

```bash
├── abi                     # runtime loading library
│  ├── Cargo.toml           # specifies dependent libraries to be used for dynamic linking
│  └── src
│     ├── import.rs         # import impls to this crate
│     ├── lib.rs            # 
│     └── main.rs           # test main: loading abi and binding interface
├── impls                   # dynamic link library 
│  └── src
│     └── lib.rs            # impls
└── README.md
```

`main` == loadl ibrary => `abi` == dynamic link ==> `impls`

设置lib类型为`cdylib`
```toml
# impls/Cargo.toml
[lib]
crate-type = ["cdylib"]
```

导出接口
```rust
// impls/src/lib.rs
#[no_mangle]
pub extern "C" fn add(left: usize, right: usize) -> usize {
    left + right
}
```



设置依赖 
```toml
# abi/Cargo.toml
[dependencies]
impls = { path = "../impls" }
```

接口定义
```rust
#[link(name = "impls")]
extern "C" {
    pub fn add(a :usize, b: usize) -> usize;
}
```

调用接口
```rust
println!("Add {}", add(1, 9));
```

## 运行时加载动态库
因为是运行时加载，所以函数的接口不需要单独定义，通常是绑定到函数指针上

```toml
# abi/Cargo.toml
[dependencies]
impls = { path = "../impls" }
libloading = "0.8.5"
```

```rust
// abi/src/main.rs
use libloading::{Library, Symbol};

static DLL_HANDLE:Lazy<Library> = Lazy::new(|| {
    unsafe {
        let dll = std::env::args().nth(1).expect("No DLL provided");
        Library::new(dll).expect("Failed to load DLL")
    }
});

// 定义好函数指针的结构体
#[derive(Debug)]
struct Interface<'a> {
    add: Symbol<'a, extern "C" fn(usize, usize) -> usize>,
    init: Symbol<'a, extern "C" fn() -> ()>,
}

// 加载动态库后，绑定函数指针
impl <'a> Interface<'a> {
    fn new() -> Self {
        unsafe {
            let add: Symbol<extern "C" fn(usize, usize) -> usize> = DLL_HANDLE.borrow().get(b"add").expect("Failed to load add");
            let init: Symbol<extern "C" fn() -> ()> = DLL_HANDLE.borrow().get(b"init").expect("Failed to load init");
            Interface {
                add,
                init
            }
        }
    }
}

fn main() {
    let handle = Interface::new();
    (handle.init)();
    let result = (handle.add)(1, 9);
    println!("Result: {}", result);
    
}

```


## Demo
项目地址 [Demo](https://github.com/kylinholmes/rs_dynamic_link)