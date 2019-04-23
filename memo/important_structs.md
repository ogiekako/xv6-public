# メモリ管理 関連

- run            - Free page の linked list (kmem が head をもつ)
- `pde_t* pgdir` - ページテーブル 

# process 関連

- ptable - proc 置き場
- proc   - プロセスの状態 (ページテーブルへのポインタ, 退避された context, 親プロセス など)
- cpu    - CPU の状態 (現在の proc や、退避された scheduler context)

# File system 関連

- bcache - buf 置き場、かつ、buf list の head を持つ。
- buf    - doubly linked list of disk contests' in memory cache.
- file   - corresponding to a file descriptor.
- inode  - ディスクにある dinode の in memory copy.
- icache - inode 置き場

