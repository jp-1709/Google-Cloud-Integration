# Investment Portfolio Management

## Google Cloud Storage (GCS) Integration Guide

------------------------------------------------------------------------

## 🚀 Getting the App (Local Development Setup)

Follow these steps to fetch and install the app locally in a Frappe
bench environment.

### 1️⃣ Navigate to Your Bench

``` bash
cd ~/your-bench-name
```

### 2️⃣ Get the App from Git Repository

``` bash
bench get-app  https://github.com/QuantbitERP/quantbit_investment_portfolio_management.git
```

### 4️⃣ Run Migration

``` bash
bench --site your-site-name migrate
bench build
bench restart
```

------------------------------------------------------------------------

# 📌 Overview

This project integrates **Google Cloud Storage (GCS)** with a Frappe
application using **S3 interoperability mode**.

Instead of storing uploaded files on the local disk, files are uploaded
directly to a **GCS bucket** using boto3.

------------------------------------------------------------------------

# 🏗 Architecture

Frappe allows overriding default file storage behavior via hooks.

We intercept:

-   `write_file`
-   `delete_file_data_content`

And redirect file operations to Google Cloud Storage.

### Components

-   `site_config.json` → Storage Configuration\
-   `requirements.txt` → Dependency Management\
-   `hooks.py` → Hook Registration\
-   `cloud.py` → Upload & Delete Logic

------------------------------------------------------------------------

# ⚙️ Configuration

## 1️⃣ site_config.json

Add the following configuration:

``` json
"file_system_storage": {
    "enabled": 1,
    "type": "s3",
    "bucket_name": "quantbit_client_1",
    "access_key": "GOOG...",
    "secret_key": "...",
    "endpoint_url": "https://storage.googleapis.com",
    "region": "asia-south1"
}
```

GCS supports the S3 API, so boto3 works seamlessly with a custom
`endpoint_url`.

------------------------------------------------------------------------

# 📦 Dependencies

Add to `requirements.txt`:

    boto3

Then run:

``` bash
bench setup requirements
```

------------------------------------------------------------------------

# 🔗 Hook Registration

Inside `hooks.py`:

``` python
write_file = "investment_portfolio_management.cloud.upload_file_to_gcs"
delete_file_data_content = "investment_portfolio_management.cloud.delete_file_from_gcs"
```

------------------------------------------------------------------------

# 🧠 Core Logic (cloud.py)

### Key Implementation Details

-   Supports polymorphic calls (`*args`, `**kwargs`)
-   Detects whether input is a `File` document or raw arguments
-   Uploads file using boto3
-   Updates `file_doc.file_url` and `file_doc.file_size` in-place
-   Sets `file_url` to HTTPS so Frappe automatically treats it as remote

⚠️ Important:\
Frappe ignores the return value of `write_file` when called through
`File.save_file()`.\
Therefore, updating the `File` document in-place is mandatory.

------------------------------------------------------------------------

# 🛠 Common Issues & Solutions

### ❌ TypeError During Upload

**Cause:** Function expected content but received File document.\
**Fix:** Handle both call signatures dynamically.

### ❌ OSError: File does not exist

**Cause:** Frappe tried validating local disk.\
**Fix:** Set `file_url` to full HTTPS URL so it's treated as remote.

### ❌ AttributeError: is_remote_file

**Cause:** Attempted manual assignment.\
**Fix:** Do not set it manually. It's derived automatically from
`file_url`.

------------------------------------------------------------------------

# ✅ Verification Checklist

### Upload

-   Upload via Frappe Desk
-   Confirm object appears in GCS bucket

### Access

-   Confirm `file_url` is public HTTPS link
-   Test direct browser access

### Delete

-   Delete File record
-   Confirm object removed from bucket

------------------------------------------------------------------------

# 🔐 Production Best Practices

-   Use IAM service accounts instead of static keys
-   Enable least privilege access policies
-   Use signed URLs for private files
-   Enable lifecycle policies for cost optimization
-   Restrict bucket-level access

------------------------------------------------------------------------

# 📅 Document Information

Generated on: 2026-02-11

------------------------------------------------------------------------

# 🎯 Conclusion

This integration provides scalable, cloud-native file storage without
modifying Frappe core code.\
By leveraging hooks and properly updating File documents in-place, the
system ensures seamless cloud storage integration.