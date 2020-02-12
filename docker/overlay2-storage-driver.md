# intro
- based on overlayfs driver reference on official docker document

# how it works
- main point: `image-layer` > `container-layer` 로 일어나는 `copy_up` operation 은 block 이 아니라 file 단위로 일어나기 때문에, file-write 와 같은 변경이 필요할때 일단 전체를 `copy_up` 해놓고 시작한다, `copy_up` 되어있는것에 후속적으로 접근하는건 괜찮지만, 최초 `copy_up` 에 대한 고려가 필요하다.

## when container reads file
it can be divided into 3 cases, `container-layer only`, `image-layer only`, `container&image layer`
- `container-layer`에만 있을땐 read file from `container-layer`, directly
- `image-layer` 에만 있을때 read file from `image-layer`, but less overhead
-  `container-layer`, `image-layer` 둘다 있을땐 read file from `container-layer` 

## when modifying files or directories
### Writing to a file for the first time
- container 가 처음에 존재하는 파일에 write 를 하는데, 실제로는 container-layer 가 아닌 image-layer 에만 있는 경우에, image-layer 에서 `copy_up` operation 해서 `image-layer` to `container-layer` 해온다. 그리고 나서 새로운 copy 에다가 write 한다. 
- block level 이 아닌 file level 에서 동작해서, file 이 크건 작건, 변경점이 크건 작건 `copy_up` operation 은 전체 file 을 복사해서 performance 에 좋지 않다. 
- 처음 `copy_up` 이후에 발생하는 같은 파일에 대한 후속작업의 오버헤드는 적다
- `AUFS`는 여러 레이어들 사이에서 파일들을 찾느라 latency 가 많이 발생했었는데, `overlayFS` 는 2개 레이어 뿐이라 성능이 낫다.
- 처음 읽을때 `overlay2` 가 `overlay` 보다 더 많은 layer 를 봐야되서 살짝 성능이 안좋으나, 결과를 캐시하기 때문에 괜찮다

> The first time a container writes to an existing file, that file does not exist in the container (upperdir).
The overlay/overlay2 driver performs a copy_up operation to copy the file from the image (lowerdir) to the container (upperdir). 
The container then writes the changes to the new copy of the file in the container layer.

> However, OverlayFS works at the file level rather than the block level. This means that all OverlayFS copy_up operations copy the entire file, even if the file is very large and only a small part of it is being modified. This can have a noticeable impact on container write performance. However, two things are worth noting: The copy_up operation only occurs the first time a given file is written to. Subsequent writes to the same file operate against the copy of the file already copied up to the container.

> OverlayFS only works with two layers. This means that performance should be better than AUFS, which can suffer noticeable latencies when searching for files in images with many layers. This advantage applies to both overlay and overlay2 drivers. overlayfs2 is slightly less performant than overlayfs on initial read, because it must look through more layers, but it caches the results so this is only a small penalty.

### deleting files and directories:

- container 에서 file 을 지우면, `whiteout file` 이 `container-layer` 에 생성되서, `image-layer` 에 있는 파일을 읽어오지 못하게 한다. 
> When a file is deleted within a container, a whiteout file is created in the container (upperdir). The version of the file in the image layer (lowerdir) is not deleted (because the lowerdir is read-only). However, the whiteout file prevents it from being available to the container.

- container 에서 directory 를 지우면, opaque directory 가 `container-layer` 에 생성되서, file 지운거랑 똑같이 `image-layer` 에 있는 디렉토리를 읽어오지 못하게 한다. 
> When a directory is deleted within a container, an opaque directory is created within the container (upperdir). This works in the same way as a whiteout file and effectively prevents the directory from being accessed, even though it still exists in the image (lowerdir).

- renaming 은 `container-layer` to `container-layer` 만 가능하다, `image-layer` 로의 rename 은 `EXDEV(cross-device link not permitted)` 에러 발생한다. 
> Renaming directories: Calling rename(2) for a directory is allowed only when both the source and the destination path are on the top layer. Otherwise, it returns EXDEV error (“cross-device link not permitted”). Your application needs to be designed to handle EXDEV and fall back to a “copy and unlink” strategy.

# do
- use `docker volume` for write-heavy workloads

>Use volumes for write-heavy workloads: Volumes provide the best and most predictable performance for write-heavy workloads. This is because they bypass the storage driver and do not incur any of the potential overheads introduced by thin provisioning and copy-on-write. Volumes have other benefits, such as allowing you to share data among containers and persisting your data even if no running container is using them.

# reference
- https://docs.docker.com/storage/storagedriver/overlayfs-driver/
