# Apple Silicon 或 無硬體支援的情境 安裝x64虛擬機
> 可以用 `docker` 跟 `gdb`

## 1. 安裝必要工具
```bash
brew install qemu cdrtools    # MacOS
scoop install qemu cdrtools   # Winodws
```

## 2. 下載虛擬機映像檔
到[Fodara官網](https://fedoraproject.org/cloud/download)下載`qcow2`格式的檔案

## 3. 擴大映像檔最大容量(預設只有20G)
```bash
qemu-img resize Fedora-Cloud-Base-Generic-42-1.1.x86_64.qcow2 100G
```

## 4. 設定環境(cloud-init)
修改 `user-data` 檔案，將 `ssh_authorized_keys` 改成自己電腦產生的 public key

## 5. 將設定檔打包成iso檔
```bash
mkisofs -output cloud-init.iso -volid cidata -joliet -rock {user-data,meta-data}
```

## 6. 啟動虛擬機
```bash
qemu-system-x86_64 \
  -name "fedora-cloud-vm" \
  -machine type=q35 \
  -accel tcg,thread=multi \
  -cpu max,check=off \
  -smp cores=4,threads=1,sockets=1 \
  -m 4096 \
  -device virtio-balloon \
  -device virtio-rng-pci \
  -drive file=Fedora-Cloud-Base-Generic-42-1.1.x86_64.qcow2,format=qcow2,if=virtio \
  -drive file=cloud-init.iso,format=raw,if=virtio,readonly=on \
  -netdev user,id=net0,hostfwd=tcp::2222-:22,hostfwd=tcp::8080-:80 \
  -device virtio-net,netdev=net0
```
