## 注册文件系统

注册和注销文件系统通过以下两个函数进行：

```cpp
// include/linux/fs.h

extern int register_filesystem(struct file_system_type *);
extern int unregister_filesystem(struct file_system_type *);
```

当文件系统挂载到指定目录后，vfs 会调用文件系统提供的 `mount()` 函数。

当路径解析到达挂载点时，就跳转到对应文件系统的根目录中。

## file_system_type

```cpp
struct file_system_type {
	const char *name;
	int fs_flags;
	int (*init_fs_context)(struct fs_context *);
	const struct fs_parameter_spec *parameters;
	struct dentry *(*mount)(struct file_system_type *, int, const char *, void *);
	void (*kill_sb)(struct super_block *);
	struct module *owner;
	struct file_system_type *next;
	struct hlist_head fs_supers;

	struct lock_class_key s_lock_key;
	struct lock_class_key s_umount_key;
	struct lock_class_key s_vfs_rename_key;
	struct lock_class_key s_writers_key[SB_FREEZE_LEVELS];

	struct lock_class_key i_lock_key;
	struct lock_class_key i_mutex_key;
	struct lock_class_key invalidate_lock_key;
	struct lock_class_key i_mutex_dir_key;
};
```

- `name`，文件系统名称，如 "ext2"、"ext4" 等。

- `fs_flags`，一些标志。

- `init_fs_context`，初始化该文件系统的 `fs_context` 对象。

- `mount`，文件系统被挂载时调用，主要任务是创建并初始化 `super_block`，返回根目录的 `dentry`。

  > `mount()` 正常返回时需要返回文件系统根路径的 `dentry`，且 `super_block` 必须处于上锁状态（确保文件系统在挂在过程中的一致性）。
  >
  > `mount()` 可以选择返回已经存在的文件系统的子树，而不是创建一个新的文件系统。比如用户将一个已经挂载的文件系统的一部分挂载到另一个位置时，或者将同一个文件系统挂载到多个不同的挂载点时，系统中只需要存在一个实际的文件系统即可。通常来说，创建 `super_block` 只是 `mount()` 的副作用。
  >
  > 第二个参数为挂载标志，如 `MS_RDONLY`，只读挂载。
  >
  > 第三个参数为设备名或路径名，具体含义取决于文件系统类型。
  >
  > - 对于块设备文件系统，通常是块设备路径，如 `/dev/sda1`。
  >
  > - 对于网络文件系统，如 NFS，可能是远程服务器名称。
  >
  > - 对于伪文件系统，如临时文件系统，可能是特殊字符串或空。
  >
  > 第四个参数为挂载选项，通常是一个字符串。
  >
  > vfs 提供了一些通用的 `mount` 实现，并提供一个 `fill_super` 回调函数来初始化 `super_block`，如 `mount_bdev()`，挂载块设备。

- `kill_sb`，文件系统关闭时调用，以销毁 `super_block`。

- `owner`，vfs 内部使用，大多数情况下应该初始化为 `THIS_MODULE`。

- `next`，vfs 内部使用，初始化为空即可。

- `fs_supers`，vfs 内部使用，管理同一文件系统的所有 `super_block` 对象。

- 剩下的是不同作用的锁。
