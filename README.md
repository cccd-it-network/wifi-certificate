# Wi-Fi Certificate GitHub Release Procedure

This repository documents the standard operational procedure for preparing, validating, and publishing new Wi-Fi/802.1X certificates and their SHA-256 checksums as GitHub Releases for deployment via Jamf Pro.

The GitHub Release object serves as the single source of truth for the certificate version deployed across macOS endpoints.

---

## Table of Contents

* [Overview & Architecture](https://www.google.com/search?q=%2523overview--architecture&utm_source=gemini)
* [Naming Conventions & Standards](https://www.google.com/search?q=%2523naming-conventions--standards&utm_source=gemini)
* [Release Procedure](https://www.google.com/search?q=%2523release-procedure&utm_source=gemini)
* [1. Prepare Working Directory](https://www.google.com/search?q=%25231-prepare-working-directory&utm_source=gemini)
* [2. Inspect and Verify Certificate](https://www.google.com/search?q=%25232-inspect-and-verify-certificate&utm_source=gemini)
* [3. Generate & Verify Checksum](https://www.google.com/search?q=%25233-generate--verify-checksum&utm_source=gemini)
* [4. Create GitHub Release & Tag](https://www.google.com/search?q=%25234-create-github-release--tag&utm_source=gemini)
* [5. Upload Assets & Add Release Notes](https://www.google.com/search?q=%25235-upload-assets--add-release-notes&utm_source=gemini)
* [6. Publish & Verify](https://www.google.com/search?q=%25236-publish--verify&utm_source=gemini)


* [Release Notes Template](https://www.google.com/search?q=%2523release-notes-template&utm_source=gemini)
* [Six-Month Rotation Lifecycle](https://www.google.com/search?q=%2523six-month-rotation-lifecycle&utm_source=gemini)
* [Jamf Pro Integration](https://www.google.com/search?q=%2523jamf-pro-integration&utm_source=gemini)

---

## Overview & Architecture

Certificates are **not** committed to the Git tree or `main` branch. Instead, they are published exclusively as binary assets attached to tagged GitHub Releases.

```text
wifi-certificate/
│
├── main branch
│   └── README.md
│
└── GitHub Releases (Tagged Commits)
    ├── v2026.09/
    │   ├── access-ise.cer
    │   └── access-ise.cer.sha256
    ├── v2027.03/
    │   ├── access-ise.cer
    │   └── access-ise.cer.sha256
    └── v2027.09/
        ├── access-ise.cer
        └── access-ise.cer.sha256

```

---

## Naming Conventions & Standards

| Component | Standard Format | Example |
| --- | --- | --- |
| **Certificate File** | `access-ise.cer` | `access-ise.cer` |
| **Checksum File** | `access-ise.cer.sha256` | `access-ise.cer.sha256` |
| **Git Tag** | `vYYYY.MM` | `v2026.09` |
| **Release Title** | `vYYYY.MM` | `v2026.09` |
| **Target Branch** | `main` | `main` |

> **Note:** Keep filenames strictly lower-case and matching exact strings. Do not rename files after generating checksums.

---

## Release Procedure

### 1. Prepare Working Directory

Obtain the newly issued certificate, place it on your workstation, and switch to a clean workspace:

```bash
mkdir -p ~/Desktop/ise-cert-release
cd ~/Desktop/ise-cert-release

```

Ensure the certificate file is correctly located:

```bash
ls -l access-ise.cer

```

### 2. Inspect and Verify Certificate

Verify that the subject, issuer, Common Name (CN), and validity dates match expectations before distribution.

For **PEM-encoded** certificates:

```bash
openssl x509 -in access-ise.cer -text -noout

```

For **DER-encoded** certificates:

```bash
openssl x509 -inform DER -in access-ise.cer -text -noout

```

**Required Checks:**

* **CN:** Matches `CN=access.ise.example.edu` (or expected wildcard/FQDN pattern).
* **Validity:** Ensure `Not Before` and `Not After` dates cover the targeted operational window.

### 3. Generate & Verify Checksum

Generate the SHA-256 hash and write it to the payload file:

```bash
shasum -a 256 access-ise.cer > access-ise.cer.sha256

```

Inspect the output to ensure formatting is clean:

```bash
cat access-ise.cer.sha256

```

*Example Output:*

```text
8f7c1234567890abcdef1234567890abcdef1234567890abcdef1234567890ab  access-ise.cer

```

Validate the file pair locally:

```bash
shasum -a 256 -c access-ise.cer.sha256

```

> **Warning:** Do not proceed if verification returns anything other than `access-ise.cer: OK`.

### 4. Create GitHub Release & Tag

1. Open the repository on GitHub.
2. Navigate to **Releases** → **Draft a new release**.
3. Click **Choose a tag**, type the new version tag (e.g., `v2026.09`), and select **Create new tag on publish**.
4. Ensure the target branch is set to `main`.
5. Set the **Release title** to match the tag exactly: `v2026.09`.

### 5. Upload Assets & Add Release Notes

Drag and drop the following two files into the **Attach binaries by dropping them here or selecting them** box:

* `access-ise.cer`
* `access-ise.cer.sha256`

Fill out the release description field using the standardized template below.

### 6. Publish & Verify

1. Review the release parameters:
* **Tag / Release Title:** `v2026.09`
* **Target:** `main`
* **Attached Assets:** `access-ise.cer` and `access-ise.cer.sha256`


2. Click **Publish release**.
3. Re-open the newly published release page to confirm both assets are visible and downloadable.

---

## Release Notes Template

Copy and fill out the template below when drafting the release notes on GitHub:

```markdown
### Wi-Fi / 802.1X Certificate Rotation

* **Certificate File:** `access-ise.cer`
* **Common Name (CN):** `access.ise.example.edu`
* **Issuer:** InCommon Intermediate CA
* **Valid From:** 2026-09-01
* **Valid Until:** 2027-03-01
* **Purpose:** Enterprise Wi-Fi 802.1X authentication
* **Target Management:** Jamf Pro

#### Checksum (SHA-256)
`8f7c1234567890abcdef1234567890abcdef1234567890abcdef1234567890ab`

```

---

## Six-Month Rotation Lifecycle

Do **not** overwrite or edit existing releases when rotating certificates. Each rotation requires a new tag and release to maintain audit logs and rollback capacity.

```text
v2026.09 (Previous)
├── access-ise.cer
└── access-ise.cer.sha256

v2027.03 (Active)
├── access-ise.cer
└── access-ise.cer.sha256

v2027.09 (Upcoming)
├── access-ise.cer
└── access-ise.cer.sha256

```

---

## Jamf Pro Integration

The client-side Jamf deployment script performs the following stateless operations upon execution:

1. Queries the GitHub API (`/releases/latest`) to determine the target tag.
2. Downloads `access-ise.cer` and `access-ise.cer.sha256` to `/private/tmp/`.
3. Validates the SHA-256 payload using native macOS utilities (`/usr/bin/shasum`).
4. Inspects the active user login keychain (`~/Library/Keychains/login.keychain-db`).
5. Purges any expired certificates matching `access.ise.*.edu`.
6. Checks if the target SHA-256 fingerprint is already present.
7. Imports the new certificate via `/usr/bin/security` only if missing.
8. Cleans up temporary files.
