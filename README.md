Wi-Fi Certificate GitHub Release Procedure
Purpose

This procedure documents how to prepare a new Wi-Fi/802.1X certificate, generate its SHA-256 checksum, and publish both files as a GitHub Release for deployment through Jamf Pro.

The GitHub Release is the source of truth for the certificate version that Jamf should deploy.

Repository Structure

The repository does not contain a Releases directory.

For example:

wifi-certificate
├── README.md
└── GitHub Releases
    ├── v2026.09
    ├── v2027.03
    └── v2027.09


The releases are GitHub Release objects associated with Git tags. The certificate files are uploaded as Release assets, not committed to the main branch.

1. Prepare the Certificate

Obtain the new certificate file.

The certificate should be named:

access-ise.cccd.edu.cer


Place the certificate in a working directory on your Mac.

For example:

mkdir -p ~/Desktop/ise-cert-release
cd ~/Desktop/ise-cert-release


Verify that the certificate is present:

ls -l access-ise.*.cer

2. Verify the Certificate

Before publishing it, inspect the certificate:

openssl x509 -in access-ise.cer -text -noout


If the certificate is DER encoded rather than PEM encoded, use:

openssl x509 -inform DER -in access-ise.cer -text -noout


Verify the important certificate information, including:

Subject / Common Name

Issuer

Valid From

Valid Until

Key information

For the Wi-Fi certificate, verify that the expected CN is present, such as:

CN=access.ise.example.edu


Also verify that the certificate has the expected expiration date.

3. Create the SHA-256 Checksum

From the directory containing access-ise.cer, run:

shasum -a 256 access-ise.cer


Example output:

8f7c1234567890abcdef1234567890abcdef1234567890abcdef1234567890ab  access-ise.cer


Create the checksum file:

shasum -a 256 access-ise.cer > access-ise.cer.sha256


Verify the file:

cat access-ise.cer.sha256


It should contain:

8f7c1234567890abcdef1234567890abcdef1234567890abcdef1234567890ab  access-ise.cer


The two files that will be uploaded to GitHub are:

access-ise.cer
access-ise.cer.sha256

4. Verify the Checksum

Before uploading the files, verify the checksum:

shasum -a 256 -c access-ise.cer.sha256


Expected result:

access-ise.cer: OK


Do not publish the certificate if the checksum verification fails.

5. Determine the Release Version

Use a version based on the certificate rotation period.

For example:

v2026.09


The next six-month rotation might be:

v2027.03


Then:

v2027.09


The v prefix is part of the tag/version.

The release name and tag should normally use the same version:

Release name: v2026.09
Tag:           v2026.09

6. Create the GitHub Release

Open the organization's GitHub repository.

Navigate to:

Releases → Draft a new release

Choose the tag

Under Choose a tag, enter:

v2026.09


If the tag does not already exist, GitHub will provide an option to create the new tag.

Create the tag from the current main branch.

The important relationship is:

main
  │
  └── commit
       │
       └── tag: v2026.09
              │
              └── GitHub Release


There is no Releases folder in the repository.

7. Name the Release

Use the same version as the tag.

For example:

v2026.09


Alternatively, if additional description is desired:

Wi-Fi Certificate v2026.09


For automation, however, keeping the release name and tag consistent is simpler.

Recommended:

Tag:          v2026.09
Release name: v2026.09

8. Upload the Certificate

In the GitHub Release page, locate the Assets section.

Drag the following two files into the Assets area:

access-ise.cer
access-ise.cer.sha256


The completed release should show:

Assets

access-ise.cer
access-ise.cer.sha256


Do not rename the files between generating the checksum and uploading them.

9. Add Release Notes

Document the certificate information in the release notes.

Example:

Wi-Fi / 802.1X Certificate Rotation

Certificate:
access-ise.cer

Certificate CN:
access.ise.example.edu

Issuer:
InCommon Intermediate CA

Valid From:
2026-09-01

Valid Until:
2027-03-01

Purpose:
Enterprise Wi-Fi 802.1X authentication

Deployment:
Jamf Pro

SHA-256:
8f7c1234567890abcdef1234567890abcdef1234567890abcdef1234567890ab


The SHA-256 value in the release notes should match the value contained in access-ise.cer.sha256.

10. Verify the Release Before Publishing

Before clicking Publish release, verify:

Release name:
v2026.09

Tag:
v2026.09

Target:
main

Assets:
access-ise.cer
access-ise.cer.sha256


Also verify:

The certificate is the intended certificate.

The CN is correct.

The issuer is correct.

The expiration date is correct.

The SHA-256 checksum was generated from the exact certificate being uploaded.

shasum -a 256 -c access-ise.cer.sha256 returns OK.

11. Publish the Release

Click:

Publish release

The release is now the current published version.

The repository will contain the release as a GitHub Release associated with the v2026.09 tag.

The certificate itself is a Release asset and is not part of the main branch.

12. Verify the Published Release

After publishing, open the release and verify that both assets are present:

v2026.09

Assets:
  access-ise.cer
  access-ise.cer.sha256


The release should also show the correct tag:

v2026.09

13. Six-Month Certificate Rotation

When the certificate needs to be replaced, do not modify the previous release.

For the next certificate:

Obtain the new certificate.

Name it access-ise.cer.

Verify the certificate.

Generate a new SHA-256 checksum.

Verify the checksum.

Create a new GitHub tag.

Create a new GitHub Release.

Upload the new access-ise.cer.

Upload the new access-ise.cer.sha256.

Add release notes.

Publish the release.

For example:

Current:

v2026.09
├── access-ise.cer
└── access-ise.cer.sha256


Six months later:

v2027.03
├── access-ise.cer
└── access-ise.cer.sha256


The old release remains available for historical reference and rollback.

14. Recommended Naming Standard

Use the following naming convention consistently:

Certificate
access-ise.cer

Checksum
access-ise.cer.sha256

Git tag
vYYYY.MM


Examples:

v2026.09
v2027.03
v2027.09
v2028.03

Release name

Use the same value as the tag:

v2026.09


This provides a simple and predictable structure for the Jamf deployment script.

15. Final Release Layout

The completed GitHub repository will conceptually contain:

Repository
│
├── main branch
│   └── README.md
│
└── GitHub Releases
    │
    ├── v2026.09
    │   ├── access-ise.cer
    │   └── access-ise.cer.sha256
    │
    ├── v2027.03
    │   ├── access-ise.cer
    │   └── access-ise.cer.sha256
    │
    └── v2027.09
        ├── access-ise.cer
        └── access-ise.cer.sha256


The Jamf script will use the latest published GitHub Release as the source of truth for the certificate version. It can compare that release version with the version currently installed on the Mac, download the corresponding certificate and checksum, verify the SHA-256 checksum, remove the old access.ise.*.edu certificates, install the new certificate, and record the deployed version.

