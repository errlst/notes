```cpp
// include/linux/fs.h

struct super_operations {
	struct inode *(*alloc_inode)(struct super_block *sb);
	void (*destroy_inode)(struct inode *);
	void (*free_inode)(struct inode *);

	void (*dirty_inode)(struct inode *, int flags);
	int (*write_inode)(struct inode *, struct writeback_control *wbc);
	int (*drop_inode)(struct inode *);
	void (*evict_inode)(struct inode *);
	void (*put_super)(struct super_block *);
	int (*sync_fs)(struct super_block *sb, int wait);
	int (*freeze_super)(struct super_block *, enum freeze_holder who);
	int (*freeze_fs)(struct super_block *);
	int (*thaw_super)(struct super_block *, enum freeze_holder who);
	int (*unfreeze_fs)(struct super_block *);
	int (*statfs)(struct dentry *, struct kstatfs *);
	int (*remount_fs)(struct super_block *, int *, char *);
	void (*umount_begin)(struct super_block *);

	int (*show_options)(struct seq_file *, struct dentry *);
	int (*show_devname)(struct seq_file *, struct dentry *);
	int (*show_path)(struct seq_file *, struct dentry *);
	int (*show_stats)(struct seq_file *, struct dentry *);
#ifdef CONFIG_QUOTA
	ssize_t (*quota_read)(struct super_block *, int, char *, size_t,
			      loff_t);
	ssize_t (*quota_write)(struct super_block *, int, const char *, size_t,
			       loff_t);
	struct dquot __rcu **(*get_dquots)(struct inode *);
#endif
	long (*nr_cached_objects)(struct super_block *,
				  struct shrink_control *);
	long (*free_cached_objects)(struct super_block *,
				    struct shrink_control *);
	void (*shutdown)(struct super_block *sb);
};
```

- `alloc_inode`，为 inode 分配内存并初始化。具体文件系统内部通常使用更大的结构包裹 `inode`，如 ext2 文件系统中：

  ```cpp
  // fs/ext2/super.c

  static struct inode *ext2_alloc_inode(struct super_block *sb)
  {
      struct ext2_inode_info *ei;
      ei = alloc_inode_sb(sb, ext2_inode_cachep, GFP_KERNEL);
      if (!ei)
          return NULL;
      ei->i_block_alloc_info = NULL;
      inode_set_iversion(&ei->vfs_inode, 1);
  #ifdef CONFIG_QUOTA
      memset(&ei->i_dquot, 0, sizeof(ei->i_dquot));
  #endif

      return &ei->vfs_inode;
  }
  ```

- `destroy_inode`，释放分配的内存。

  > 默认实现会调用 `call_rcu()`。
  >
  > ```cpp
  > // fs/inode.c
  >
  > static void destroy_inode(struct inode *inode)
  > {
  > 	const struct super_operations *ops = inode->i_sb->s_op;
  >
  > 	BUG_ON(!list_empty(&inode->i_lru));
  > 	__destroy_inode(inode);
  > 	if (ops->destroy_inode) {
  > 		ops->destroy_inode(inode);
  > 		if (!ops->free_inode)
  > 			return;
  > 	}
  > 	inode->free_inode = ops->free_inode;
  > 	call_rcu(&inode->i_rcu, i_callback);
  > }
  > ```

- `free_inode`，释放分配的内存，但只有在 `destroy_inode()` 中调用 `call_rcu()` 时，才会调用 `free_inode`。

- `dirty_inode`，将 inode 标记为 dirty。dirty 表示 inode 数据被修改，但还未同步回磁盘中。

  > vfs 通过调用 `mark_inode_dirty()` 和 `mark_inode_dirty_sync()` 将 inode 标记为 dirty。

- `write_inode`，将 inode 写入磁盘。
