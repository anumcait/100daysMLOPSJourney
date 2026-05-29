# Day 13 — Pull DVC-Tracked Data from Remote

A new xFusionCorp Industries team member has cloned the fraud-detection repository onto a fresh machine. The DVC remote is already configured to point at the team's SeaweedFS bucket, but dvc pull is failing. Diagnose the cause, correct the configuration, and pull the dataset. 

A cloned project exists at /root/code/fraud-detection/ with DVC initialised, the data/raw/transactions.csv.dvc pointer file present, but the dataset itself missing from disk and from the local DVC cache. 

SeaweedFS is already running on the controlplane and the dataset has already been pushed to the dvc-storage bucket—open the SeaweedFS Filer button at the top of the lab and navigate to /buckets/dvc-storage/ to confirm that the object is there. 

S3 endpoint: http://localhost:8333 
Credentials: weedadmin / weedadmin123 
Review .dvc/config and correct everything that prevents dvc pull from authenticating against SeaweedFS. 

After the fix, the s3 remote must use: 
The access key (access_key_id) weedadmin 
The secret key (secret_access_key) weedadmin123. 
Pull the dataset. 
After the pull, data/raw/transactions.csv must be present on disk and its content must match the hash recorded in the .dvc pointer.

## 📋 Task Summary

A cloned project exists at `/root/code/fraud-detection/` with DVC initialized, the `data/raw/transactions.csv.dvc` pointer file present, but the dataset itself missing from disk and from the local DVC cache.

SeaweedFS is running on the controlplane and the dataset has already been pushed to the `dvc-storage` bucket.

**S3 Details:**
- **Endpoint:** `http://localhost:8333`
- **Credentials:** `weedadmin` / `weedadmin123`

The goal is to correction the configuration to authenticate against SeaweedFS and successfully pull the dataset.

---

## 🛠️ Step 1: Diagnose the Failure

First, navigate to the project directory and attempt to pull the data to see the error.

```bash
cd /root/code/fraud-detection

# Try to pull (will likely fail due to missing credentials)
dvc pull
```

---

## 🛠️ Step 2: Configure Local Credentials and Endpoint

Since credentials should not be committed to the repository, use the `--local` flag to store them in `.dvc/config.local`.

```bash
# Set access key
dvc remote modify --local s3 access_key_id weedadmin

# Set secret key
dvc remote modify --local s3 secret_access_key weedadmin123

# Set custom endpoint URL
dvc remote modify --local s3 endpointurl http://localhost:8333

# Disable SSL (since we are using localhost http)
dvc remote modify --local s3 use_ssl false
```

---

## 🛠️ Step 3: Verify Local Configuration

Check the `.dvc/config.local` file to ensure the settings are correctly applied.

```bash
cat .dvc/config.local
```

### Expected `.dvc/config.local`
```ini
['remote "s3"']
    access_key_id = weedadmin
    secret_access_key = weedadmin123
    endpointurl = http://localhost:8333
    use_ssl = false
```

---

## 🛠️ Step 4: Pull the Dataset

Now that authentication is configured, pull the data from the SeaweedFS remote.

```bash
dvc pull -v
```

---

## ✅ Step 5: Verify the Data

Confirm that the file is present on disk and its content is accessible.

```bash
# List the file
ls -l data/raw/transactions.csv

# Peek at the data
head data/raw/transactions.csv
```

---

## 🧠 Key Concepts Learned

| Concept | Detail |
|---------|--------|
| **`dvc pull`** | Downloads tracked data from remote storage to the local cache and links it to the workspace. |
| **`.dvc/config.local`** | A file used to store local-only configuration (like credentials) that is ignored by Git. |
| **`--local` Flag** | Tells DVC to save the configuration change in `.dvc/config.local` instead of the project-shared `.dvc/config`. |
| **`use_ssl false`** | Necessary when connecting to an S3-compatible service that does not use HTTPS (common in local labs/dev environments). |

---


## ⚠️ Common Pitfalls

1. **Hardcoding Credentials** — Never run `dvc remote modify s3 access_key_id ...` without `--local` unless you want your keys committed to Git!
2. **Missing Endpoint** — Standard DVC assumes AWS S3. For SeaweedFS/MinIO, the `endpointurl` is mandatory.
3. **Cache Mismatch** — If `dvc pull` says "Everything is up to date" but the file is missing, check if the `.dvc` pointer file matches the remote hash.

---

### Screenshots

<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/bbad7bc8-ce00-4a55-a065-13b0bb996a1d" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/46922672-18ec-4f86-a7ac-e10bb4993c42" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/1bbf3ece-5a26-48be-8af6-0e352a5d002f" />


