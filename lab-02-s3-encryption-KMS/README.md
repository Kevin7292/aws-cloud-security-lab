# Lab 02 — S3 Encryption with KMS | AWS Cloud Security Portfolio

## Part 1: Create the IAM Role

Created an IAM role named `lab02-s3-kms-role` with `AmazonS3ReadOnlyAccess` attached.

This role will be added to the KMS key policy in the next step, making it the only identity authorized to encrypt and decrypt objects in the S3 bucket. Using a role instead of a user follows the principle of least privilege — the role can be assumed by a specific service or EC2 instance rather than granting permissions to a person's static credentials.

![Role Created](./screenshots/1_RoleCreated.png)

---

## Part 2: Create the Restricted IAM User

Created an IAM user named `lab02-restricted-user` with `AmazonS3ReadOnlyAccess` attached.

This user has permission to list and read S3 objects but will NOT be added to the KMS key policy. The purpose is to demonstrate that S3 read access is not enough when objects are encrypted with a customer-managed KMS key — KMS enforces its own access control layer separately from S3.

![IAM User Creation](./screenshots/2_IAMUserCreation.png)

---

## Part 3: Create a Custom KMS Key

Created a symmetric KMS key with the alias `lab02-s3-key`.

During key creation, I configured the key usage permissions to include `lab02-s3-kms-role` as an authorized user of the key. The restricted user was intentionally left out.

The key policy controls who can call `kms:GenerateDataKey` (to encrypt) and `kms:Decrypt` (to decrypt). These are separate from any S3 bucket policy or IAM policy. If an identity is not listed in the key policy, KMS will deny the request regardless of what S3 allows.

![Custom KMS Key Created](./screenshots/3_CreateCustomKey.png)

---

## Part 4: Create the S3 Bucket with SSE-KMS Default Encryption

Created an S3 bucket named `lab02-kms-encryption-test-kmg` in `us-east-2` with default encryption set to SSE-KMS using `lab02-s3-key`.

With this configuration, every object uploaded to the bucket is automatically encrypted using the custom KMS key. There is no option to upload an unencrypted object — encryption is enforced at the bucket level.

| Setting | Value |
|---------|-------|
| Bucket name | `lab02-kms-encryption-test-kmg` |
| Region | `us-east-2` |
| Default encryption | SSE-KMS |
| KMS key | `lab02-s3-key` |
| Block public access | Enabled |

![S3 Bucket with KMS Encryption](./screenshots/4_S3withKMS.png)

---

## Part 5: Upload a Test File and Confirm Encryption

Created a file named `secret-file.txt` and uploaded it to the bucket.

After upload, I opened the object properties and confirmed that the file was encrypted using the custom KMS key. The KMS key ARN is visible in the object's encryption details, confirming SSE-KMS was applied at upload time.

![File Uploaded](./screenshots/5_FileUploaded.png)

![File Properties Showing KMS Encryption](./screenshots/5_FileProperties.png)

---

## Part 6: Confirm Access Denied for Restricted User

Signed in as `lab02-restricted-user` in a separate browser window and navigated to the S3 bucket.

The restricted user could see the file listed in the bucket — their `AmazonS3ReadOnlyAccess` policy allows `s3:GetObject`. However, when attempting to open or download `secret-file.txt`, the request failed.

The reason: before S3 can serve the object, it must call KMS to decrypt it. KMS checks the key policy, finds that `lab02-restricted-user` is not listed as an authorized identity, and returns an access denied error. The decrypt call never completes, and S3 cannot serve the file.

The error message confirmed this exactly:

> `lab02-restricted-user is not authorized to perform: kms:Decrypt on resource: arn:aws:kms:us-east-2:...:key/...`

This is the key concept of the lab: **S3 permissions and KMS permissions are independent layers.** Having S3 read access is not sufficient when the object is encrypted with a customer-managed KMS key. Both layers must allow the identity.

![File Visible But Access Denied](./screenshots/6_FileDisplayed.png)

![Access Denied Error from KMS](./screenshots/6_AccessDenied.png)

---

## Part 7: Enable S3 Versioning

Enabled versioning on the bucket from the **Properties** tab.

With versioning enabled, S3 stores a separate version of every object each time it is uploaded. I uploaded a modified version of `secret-file.txt` with different content to create a second version. Both versions are stored independently with unique version IDs.

In a real environment, versioning protects against accidental deletion and overwrites. Combined with KMS encryption, each version is encrypted and access-controlled independently.

![S3 Versioning Enabled with Two Versions](./screenshots/7_S3Versioning.png)

---

## Part 8: Create a Lifecycle Rule for Glacier

Created a lifecycle rule named `lab02-glacier-transition` to transition all current object versions to **S3 Glacier Flexible Retrieval** after 30 days.

| Setting | Value |
|---------|-------|
| Rule name | `lab02-glacier-transition` |
| Scope | All objects in the bucket |
| Transition | S3 Glacier Flexible Retrieval |
| Days after creation | 30 |

S3 Glacier costs approximately $0.004/GB/month compared to $0.023/GB/month for S3 Standard — about an 80% cost reduction for data that is retained for compliance but rarely accessed. This pattern is common in regulated industries where data must be kept for years but does not need to be immediately available.

![Glacier Lifecycle Rule](./screenshots/8_GlacierRule.png)

---

## Part 9: Cleanup

Deleted all resources in dependency order to avoid charges.

1. Emptied and deleted the S3 bucket — removes all object versions
2. Scheduled KMS key deletion with a 7-day waiting period — KMS enforces a minimum waiting period before permanent deletion
3. Deleted the IAM user `lab02-restricted-user`
4. Deleted the IAM role `lab02-s3-kms-role`

![KMS Key Scheduled for Deletion](./screenshots/10_KeyDeletion.png)

![IAM User Deleted](./screenshots/10_DeleteUser.png)

![IAM Role Deleted](./screenshots/10_DeleteRole.png)

---

## Key Concepts Demonstrated

- SSE-KMS encryption enforced at the S3 bucket level
- Customer-managed KMS keys vs AWS-managed keys
- KMS key policies as an independent access control layer separate from IAM and S3 bucket policies
- Defense in depth: both S3 permissions AND KMS key policy must allow access
- S3 versioning for data protection against accidental deletion or overwrite
- S3 lifecycle rules for automated cost-optimized archiving to Glacier
- Least privilege: KMS key usage scoped to a specific IAM role only
- Secure cleanup including KMS key deletion scheduling
