实现对系统调用的追踪。

为`TaskManager`添加追踪信息：

```rust
/// Inner of Task Manager

pub struct TaskManagerInner {

    /// task list

    tasks: [TaskControlBlock; MAX_APP_NUM],

    /// id of current `Running` task

    current_task: usize,

  

    task_info_map: [TaskInfo; MAX_APP_NUM],

}
```

添加初始化：
```rust
let task_info_map = [TaskInfo {syscall_count: [0; MAX_SYSCALL_NUM],}; MAX_APP_NUM];
```

```rust
/// syscall count
pub const MAX_SYSCALL_NUM: usize = 1024;
#[derive(Copy, Clone)]
pub struct TaskInfo {
	pub syscall_count: [usize; MAX_SYSCALL_NUM],
}
```

添加接口
```rust
    fn inc_current_syscall_count(&self, syscall_id: usize) {

        let mut inner = self.inner.exclusive_access();

        let current = inner.current_task;

        inner.task_info_map[current].syscall_count[syscall_id] += 1;

    }

  

    fn get_current_syscall_count(&self, syscall_id: usize) -> usize {

        let inner = self.inner.exclusive_access();

        let current = inner.current_task;

        inner.task_info_map[current].syscall_count[syscall_id]

    }

  

}
```

实现systrace:
```rust
pub fn sys_trace(_trace_request: usize, _id: usize, _data: usize) -> isize {

    trace!("kernel: sys_trace");

  

    match _trace_request {

        0 => {

            let addr = _id as *const u8;

            let data = unsafe { *addr };

            data as isize

        }

  

        1 => {

            let addr = _id as *mut u8;

            unsafe { *addr = _data as u8 }

            0

        }

  

        2 => {

            get_current_syscall_count(_id) as isize

        }

  

        _ => -1,

    }

}
```

注册syscall
```rust
        SYSCALL_TRACE => {

            sys_trace(args[0], args[1], args[2])

        }
```

