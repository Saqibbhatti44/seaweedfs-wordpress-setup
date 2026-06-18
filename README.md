# SeaweedFS + WordPress Local Setup Guide  Prepared By SAQIB SAJID | ATL AT DIGITAL GRAVITY

A complete step-by-step guide to connect **WordPress** with **SeaweedFS** on your local machine using XAMPP and the **Advanced Media Offloader** plugin.

Instead of storing media files in `wp-content/uploads`, all WordPress uploads go directly to SeaweedFS — giving you a local S3-compatible object storage experience.

---

## What You Need

- Windows PC
- XAMPP installed (WordPress running locally)
- [SeaweedFS](https://github.com/seaweedfs/seaweedfs) `weed.exe`
- [Advanced Media Offloader](https://wordpress.org/plugins/advanced-media-offloader/) WordPress plugin

---

## Architecture

```
Browser → WordPress (XAMPP) → Advanced Media Offloader → SeaweedFS S3 Gateway → wordpress-media bucket
```

SeaweedFS runs all components in one command:

| Component | Port |
|---|---|
| Master server | 9333 |
| Volume server | 8080 |
| Filer | 8888 |
| S3 gateway | 8333 |
| Admin UI | 23646 |

---

## Step 1 — Download SeaweedFS

1. Go to [SeaweedFS Releases](https://github.com/seaweedfs/seaweedfs/releases)
2. Download the latest `windows_amd64.tar.gz`
3. Extract and place `weed.exe` at:

```
D:\wampp\seaweedfs\weed.exe
```

---

## Step 2 — Create the Data Folder

Open Command Prompt and run:

```cmd
mkdir D:\wampp\seaweedfs\data
```

---

## Step 3 — Start SeaweedFS

```cmd
D:\wampp\seaweedfs\weed.exe mini -dir=D:\wampp\seaweedfs\data -s3.port=8333
```

> ⚠️ Keep this CMD window open. Closing it stops SeaweedFS.

Verify it is running by opening: `http://localhost:9333`

---

## Step 4 — Create S3 Credentials

Open a **second CMD window** and run:

```cmd
setx AWS_ACCESS_KEY_ID "weed-access-key"
setx AWS_SECRET_ACCESS_KEY "weed-secret-key"
```

Then open the **Admin UI** at `http://localhost:23646`:

1. Click **Users** → **Add User**
2. Enter username: `weedadmin`
3. Select permissions: **S3 Admin (Full Access)**
4. Leave **Generate access key automatically** checked
5. Click **Create User**
6. **Copy and save the generated Access Key and Secret Key**

---

## Step 5 — Create the Bucket

In Admin UI:

1. Click **Buckets** → **Create New S3 Bucket**
2. Enter bucket name: `wordpress-media`
3. Click **+ Create Bucket**

---

## Step 6 — Install the WordPress Plugin

In your WordPress Admin:

1. Go to **Plugins → Add New**
2. Search for `Advanced Media Offloader`
3. Click **Install Now** → **Activate**

---

## Step 7 — Configure the Plugin

Go to **Settings → Advanced Media Offloader** and fill in:

| Field | Value |
|---|---|
| Cloud Provider | Any S3-Compatible Storage |
| Provider Name | SeaweedFS Local |
| Access Key ID | *(your generated key)* |
| Secret Access Key | *(your generated secret)* |
| S3 Endpoint URL | `http://localhost:8333` |
| Region | `us-east-1` |
| Use Path-Style Endpoint | ✅ Checked |
| Bucket Name | `wordpress-media` |
| Custom Domain | *(leave empty)* |
| Retention Policy | Retain Local Files |

Click **Save Credentials** → then **Test Connection** → should show ✅ Connected

---

## Step 8 — Fix Public Access

In Admin UI → **Buckets** → **wordpress-media** → add this bucket policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::wordpress-media/*"
    }
  ]
}
```

---

## Step 9 — Test It

1. Go to **WordPress Media → Add New**
2. Upload any image
3. Check the file URL — it should now show:

```
✅ http://localhost:8333/wordpress-media/2026/06/your-image.jpg
```

instead of:

```
❌ http://your-site/wp-content/uploads/2026/06/your-image.jpg
```

---

## ⚠️ Every Time You Restart Your PC

SeaweedFS does not start automatically. Run this command every time:

```cmd
D:\wampp\seaweedfs\weed.exe mini -dir=D:\wampp\seaweedfs\data -s3.port=8333
```

> 💡 Tip: Create a `.bat` file with this command so you can launch it with one double-click.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Plugin can't connect | Make sure `weed.exe` is still running in CMD |
| AccessDenied on file URL | Add the public bucket policy from Step 8 |
| Uploads still going to wp-content | Check plugin is set to "Auto-Offload on Upload" |
| Port 8333 not working | Check Windows Firewall is not blocking it |

---

## Download

A full Word document version of this guide is included in this repository:
📄 `SeaweedFS_WordPress_Setup_Guide.docx`

---

## Resources

- [SeaweedFS GitHub](https://github.com/seaweedfs/seaweedfs)
- [Advanced Media Offloader Plugin](https://wordpress.org/plugins/advanced-media-offloader/)
- [SeaweedFS Documentation](https://github.com/seaweedfs/seaweedfs/wiki)

---

## License

MIT — free to use, share, and modify.
