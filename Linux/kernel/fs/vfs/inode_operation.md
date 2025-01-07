`inode_operation` 描述 vfs 如何在文件系统中操作 inode。

```cpp
// inlcude/linux/fs.h

struct inode_operations {
	struct dentry *(*lookup)(struct inode *, struct dentry *, unsigned int);
	const char *(*get_link)(struct dentry *, struct inode *,
				struct delayed_call *);
	int (*permission)(struct mnt_idmap *, struct inode *, int);
	struct posix_acl *(*get_inode_acl)(struct inode *, int, bool);

	int (*readlink)(struct dentry *, char __user *, int);

	int (*create)(struct mnt_idmap *, struct inode *, struct dentry *,
		      umode_t, bool);
	int (*link)(struct dentry *, struct inode *, struct dentry *);
	int (*unlink)(struct inode *, struct dentry *);
	int (*symlink)(struct mnt_idmap *, struct inode *, struct dentry *,
		       const char *);
	int (*mkdir)(struct mnt_idmap *, struct inode *, struct dentry *,
		     umode_t);
	int (*rmdir)(struct inode *, struct dentry *);
	int (*mknod)(struct mnt_idmap *, struct inode *, struct dentry *,
		     umode_t, dev_t);
	int (*rename)(struct mnt_idmap *, struct inode *, struct dentry *,
		      struct inode *, struct dentry *, unsigned int);
	int (*setattr)(struct mnt_idmap *, struct dentry *, struct iattr *);
	int (*getattr)(struct mnt_idmap *, const struct path *, struct kstat *,
		       u32, unsigned int);
	ssize_t (*listxattr)(struct dentry *, char *, size_t);
	int (*fiemap)(struct inode *, struct fiemap_extent_info *, u64 start,
		      u64 len);
	int (*update_time)(struct inode *, int);
	int (*atomic_open)(struct inode *, struct dentry *, struct file *,
			   unsigned open_flag, umode_t create_mode);
	int (*tmpfile)(struct mnt_idmap *, struct inode *, struct file *,
		       umode_t);
	struct posix_acl *(*get_acl)(struct mnt_idmap *, struct dentry *, int);
	int (*set_acl)(struct mnt_idmap *, struct dentry *, struct posix_acl *,
		       int);
	int (*fileattr_set)(struct mnt_idmap *idmap, struct dentry *dentry,
			    struct fileattr *fa);
	int (*fileattr_get)(struct dentry *dentry, struct fileattr *fa);
	struct offset_ctx *(*get_offset_ctx)(struct inode *inode);
} ____cacheline_aligned;
```

- `create`，被 `open(2)` 和 `creat(2)` 调用。
