
## Troubleshooting

### `TypeError: Eip._internal_init() got an unexpected keyword argument 'vpc'`

**Cause:** The `vpc=True` argument was removed in `pulumi-aws` v7.

**Fix:** Replace `vpc=True` with `domain="vpc"`:

```python
# Before (broken in v7+)
eip = ec2.Eip('nat-eip', vpc=True)

# After
eip = ec2.Eip('nat-eip', domain="vpc")
```

---

### `error: unable to validate credentials`

**Cause:** AWS credentials are missing or expired in your environment.

**Fix:** Re-configure credentials:

```bash
aws configure
# or export directly
export AWS_ACCESS_KEY_ID=your_key
export AWS_SECRET_ACCESS_KEY=your_secret
export AWS_DEFAULT_REGION=us-east-1
```

Verify they work:

```bash
aws sts get-caller-identity
```

---

### `pulumi:pulumi:Stack ... create error: Program failed with an unhandled exception`

**Cause:** A Python runtime error in `__main__.py` (e.g., wrong argument, missing import, or API change).

**Fix:** Read the full traceback in the diagnostics output — it will point to the exact line and error. Common causes:

- Outdated argument names (check the [Pulumi AWS changelog](https://github.com/pulumi/pulumi-aws/blob/master/CHANGELOG.md))
- Missing `pulumi_aws` version pinned in `requirements.txt`
- Using deprecated resource classes (e.g., `s3.Bucket` instead of `s3.BucketV2`)

---

### `[remote rejected] master (refusing to delete the current branch)`

**Cause:** You're trying to delete the default branch on GitHub/GitLab.

**Fix:** Change the default branch first in your repo settings:

```
GitHub: Settings → Branches → Default Branch → switch to main → Update
```

Then re-run:

```bash
git push origin --delete master
```

---

### `Http response code: NotFound from 'POST https://api.github.com/actions/runner-registration'`

**Cause:** The GitHub Actions runner registration token is expired or invalid. Tokens expire within ~1 hour of generation.

**Fix:** Dynamically fetch a fresh token in your workflow instead of hardcoding it:

```bash
TOKEN=$(curl -s -X POST \
  -H "Authorization: token $GH_PAT" \
  -H "Accept: application/vnd.github+json" \
  https://api.github.com/repos/<owner>/<repo>/actions/runners/registration-token \
  | jq -r .token)

./config.sh --url https://github.com/<owner>/<repo> --token $TOKEN --unattended
```

Store your PAT (classic, with `repo` scope) as a secret named `GH_PAT`.

---

### `sudo: ./svc.sh: command not found`

**Cause:** The `svc.sh` script is being called from the wrong directory. It only exists inside the `actions-runner/` folder.

**Fix:** Make sure you `cd` into the runner directory before calling it:

```bash
cd ~/actions-runner
sudo ./svc.sh install
sudo ./svc.sh start
```

---

