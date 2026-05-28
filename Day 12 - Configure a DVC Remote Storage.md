# Day 12 — Configure a DVC Remote Storage

The xFusionCorp Industries ML team uses SeaweedFS as the shared S3-compatible object store for DVC-tracked data. A .dvc/config already declares a remote called s3 for the fraud-detection project, but dvc push currently fails. Correct the configuration and push the tracked data into the SeaweedFS bucket. 

A project exists at /root/code/fraud-detection/ with DVC initialised and data/raw/transactions.csv already tracked. 

SeaweedFS is already running on the controlplane: 

S3 endpoint: http://localhost:8333 
Filer UI: open the SeaweedFS Filer button at the top of the lab (forwarded port 8888) – buckets are visible under /buckets/. Credentials: weedadmin / weedadmin123 (already set in .dvc/config) Bucket name: dvc-storage (already created and visible in the Filer UI under /buckets/dvc-storage) Review the existing .dvc/config and correct everything that prevents dvc push from succeeding. The remote called s3 must: 

point at the dvc-storage bucket using s3://; 

use the correct SeaweedFS S3 endpoint URL; 

be marked as the default remote. 

Push the tracked data. After the push, the dvc-storage bucket in the SeaweedFS Filer UI must contain at least one object under the files/md5/... prefix.

## 📋 Task Summary

A project exists at `/root/code/fraud-detection/` with DVC initialised and `data/raw/transactions.csv` already tracked. SeaweedFS is running on the controlplane with an S3 endpoint at `http://localhost:8333`. The goal is to:
1. Point the `s3` remote to the `dvc-storage` bucket using `s3://`.
2. Configure the correct SeaweedFS S3 endpoint URL.
3. Mark the `s3` remote as the default remote.
4. Push the tracked data to the remote storage.

---

## 🛠️ Step 1: Verify Current Configuration

First, check the existing DVC configuration to identify what needs to be changed.

```bash
cd /root/code/fraud-detection

# Verify current config
cat .dvc/config
```

---

## 🛠️ Step 2: Correct the Remote URL

The current remote URL might be incorrect or missing the bucket name. Point it to the `dvc-storage` bucket.

```bash
dvc remote modify s3 url s3://dvc-storage
```

---

## 🛠️ Step 3: Configure the SeaweedFS S3 Endpoint

Since SeaweedFS is an S3-compatible store (not AWS S3), DVC needs to know the custom endpoint URL.

```bash
dvc remote modify s3 endpointurl http://localhost:8333
```

---

## 🛠️ Step 4: Set the Default Remote

Ensure that `dvc push` and `dvc pull` use the `s3` remote by default.

```bash
dvc remote default s3
```

---

## 🛠️ Step 5: Push Tracked Data

Upload the data from the local cache to the configured SeaweedFS remote storage.

```bash
dvc push
```

---

## ✅ Step 6: Verify the Setup

Check the remote list and the final configuration file to ensure everything was applied correctly.

```bash
# List remotes
dvc remote list

# Check config file
cat .dvc/config

# Verify status (should be up to date)
dvc status -c
```

### Expected `.dvc/config`
```ini
['remote "s3"']
    url = s3://dvc-storage
    endpointurl = http://localhost:8333
    access_key_id = weedadmin
    secret_access_key = weedadmin123
[core]
    remote = s3
```

---

## 🧠 Key Concepts Learned

| Concept | Detail |
|---------|--------|
| **DVC Remote** | A storage location (S3, GCS, Azure, SSH, etc.) where DVC-tracked data is stored and shared. |
| **S3-Compatible Storage** | Services like SeaweedFS, MinIO, or Ceph that implement the S3 API, requiring a custom `endpointurl`. |
| **Default Remote** | Setting a default remote allows you to run `dvc push` and `dvc pull` without specifying the remote name. |
| **`dvc remote modify`** | Command used to update specific parameters (url, endpoint, credentials) of an existing remote. |
| **`dvc push`**| Uploads tracked files from the local `.dvc/cache` to the remote storage. |

---

## ⚠️ Common Pitfalls

1. **Incorrect Endpoint Protocol** — Forgetting `http://` or `https://` in the `endpointurl` will cause connection failures.
2. **Missing Bucket Prefix** — The `url` must start with the schema (e.g., `s3://`) followed by the bucket name.
3. **Credentials Placement** — For security, credentials like `access_key_id` are often stored in `.dvc/config.local` (not committed to Git), but in this lab environment they are in the project config.
4. **Not Setting Default** — If no default remote is set, `dvc push` will fail unless you run `dvc push -r s3`.
