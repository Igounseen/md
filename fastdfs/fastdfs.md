## FastDFS

**Run as a tracker**

```bash
docker run -ti -d --name trakcer -v ~/tracker_data:/fastdfs/tracker/data --net=host season/fastdfs tracker
```



>tracker 默认端口：22122
>
>映射路径 /fastdfs/tracker/data 保存数据


**Run as a storage**

```
docker run -ti --name storage -v ~/storage_data:/fastdfs/storage/data -v ~/store_path:/fastdfs/store_path --net=host -e TRACKER_SERVER:192.168.1.2:22122 season/fastdfs storage
```

>storage_data:  equal to "base_path" in store.conf
>
>store_path: equal to "store_path0" in store.conf
>
>TRACKER_SERVER: tracker address

**Run as a shell**

```
docker run -ti --name fdfs_sh --net=host season/fastdfs sh
```

