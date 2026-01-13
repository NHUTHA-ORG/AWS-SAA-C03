
## 📊 STORAGE PROTOCOLS × OS SUPPORT × AWS SERVICES (SAA-C03)

| Protocol         | Storage type  | OS support                   | AWS Services                                                                   | **BE (Backend storage)**                             | Exam keywords              |
| ---------------- | ------------- | ---------------------------- | ------------------------------------------------------------------------------ | ---------------------------------------------------- | -------------------------- |
| **NFS**          | File          | Linux ✅, macOS ✅, Windows ⚠️ | **EFS**, **S3 File Gateway**, **FSx for Lustre**, **FSx for NetApp ONTAP**     | EFS native storage, **S3**, Lustre FS, ONTAP volumes | POSIX, shared FS           |
| **SMB**          | File          | Windows ✅, Linux ✅, macOS ✅  | **FSx for Windows File Server**, **S3 File Gateway**, **FSx for NetApp ONTAP** | FSx Windows storage, **S3**, ONTAP volumes           | Windows file share, AD     |
| **iSCSI**        | Block         | Linux ✅, Windows ✅, macOS ⚠️ | **Storage Gateway – Volume Gateway**, **FSx for NetApp ONTAP**                 | **S3** (gateway), ONTAP block volumes                | Block storage, low latency |
| **HTTP / HTTPS** | Object        | All OS ✅                     | **Amazon S3**, **CloudFront**                                                  | **S3**                                               | Object storage, REST       |
| **SFTP / FTPS**  | File transfer | All OS ✅                     | **AWS Transfer Family**                                                        | **S3 / EFS**                                         | Secure file transfer       |
| **Lustre**       | File (HPC)    | Linux ✅ only                 | **FSx for Lustre**                                                             | Lustre FS + optional **S3 integration**              | HPC, high throughput       |

---

## 🧠 MEMORY HOOK (5 GIÂY TRƯỚC KHI CHỌN ĐÁP ÁN)

> **Linux → NFS → EFS**
> **Windows → SMB → FSx Windows**
> **Block → iSCSI → Volume Gateway**
> **Object → HTTP → S3**

---

## ⚠️ EXAM TRAPS

❌ S3 không mount trực tiếp NFS/SMB
❌ Block storage không share multi-host
❌ Windows app → tránh EFS

