# Host the custom store for test02

This archive contains the prepared STEP3 SIM2 catalog and all ten download
parts. Extract this outer archive once. Keep `dbs.z` and the `.zip.1` through
`.zip.10` files as they are. The original bank bytes are unchanged.

## Required layout

The folder containing these paths is the Store URL root:

| Path relative to the root | Purpose |
| --- | --- |
| `dbs.z` | Catalog archive; leave compressed |
| `dbs.md5` | Catalog checksum; leave unchanged |
| `Icons/uworld.png` | Store icon |
| `banks/Parts/STEP3-2026-Oct-SIM2-v202612-e9ff4ad7730a.zip.N` | Ten parts, with N from 1 through 10 |
| `banks/Parts/STEP3-2026-Oct-SIM2-v202612-e9ff4ad7730a.zip.N.md5` | One checksum for each part |

There are 23 required store files. Names and capital letters matter, including
`Icons` and `Parts`. The additional `index.html`, `404.html`, `.nojekyll`, and
this README help with GitHub Pages hosting. They are not needed by the app.

## GitHub Pages

1. Create a repository, for example `imd-store`. GitHub Free requires a public
   repository for Pages.
2. Extract this archive and upload its contents into the repository root on
   `main`. Preserve the `Icons` and `banks/Parts` folders. Upload the extracted
   files, not this outer ZIP. Keep `.nojekyll`; if your uploader omits it,
   create an empty file with that name at the repository root.
3. Open **Settings > Pages**. Choose **Deploy from a branch**, select **main**
   and **/ (root)**, then **Save**.
4. Wait for deployment and copy the published URL from Pages settings.
   For this example it is `https://USERNAME.github.io/imd-store`.

The app needs the published Pages URL. A `github.com/.../blob/...` link serves
a GitHub interface, not the raw store files.

## Cloudflare R2: dashboard upload

Use R2 for the Cloudflare option. Cloudflare Pages currently documents HTTP
200 responses to range requests; the downloader needs correct byte ranges
for reliable resume. R2 documents range support for object reads. Verify the
actual public URL with the range check below after uploading.

1. Open **Storage & databases > R2 > Overview** in the Cloudflare dashboard.
   Enable R2 if prompted, then create a Standard storage bucket, for example
   `imd-store-test`. R2 setup includes a subscription checkout with included
   free usage; check the account's displayed terms.
2. Use **Create folder** to create `Icons`, and then `banks` with `Parts`
   inside it. Upload `uworld.png` inside `Icons`. Upload all 20 files from the
   extracted `banks/Parts` folder inside the bucket's matching folder.
3. Upload `dbs.z` and `dbs.md5` at the bucket root. No enclosing ZIP or extra
   parent folder is needed. The four GitHub helper files are optional here.
4. In bucket **Settings > Public Development URL**, choose **Enable** and
   confirm by typing `allow`. Copy the resulting HTTPS public URL.
   This exposes the bucket's objects publicly; the `r2.dev` endpoint is
   intended for testing and is rate-limited.

Use the `https://pub-....r2.dev` URL shown by Cloudflare in the app, not the
dashboard URL or the S3 API endpoint. The bucket root need not show a webpage;
test the `/dbs.md5` object instead.

## Connect the app

Use **IMD_CUSTOM_STORE_test02.apk** (`net.imedicaldoctor.imd.test02`). Open
**Settings > Custom Store URL**, paste the HTTPS root URL, and choose
**Save and close**. Reopen the app and open Store. No new APK is needed for
a different host. The original test01 build does not have this setting.

The URL should point to the folder containing `dbs.z`, without `/dbs.z`
appended. Include the repository path when using GitHub Pages.

## Check the deployed files

Open `YOUR_STORE_URL/dbs.md5`. Its exact content should be:

`cayQZ/N/htEwgGCX3tvzGQ==`

For a resume check, run this GET request with your actual Store URL:

```sh
STORE_BASE='https://USERNAME.github.io/imd-store'
curl --fail --silent --show-error --dump-header - --output range-test.bin \
  --header 'Accept-Encoding: identity' --header 'Range: bytes=0-99' \
  "$STORE_BASE/banks/Parts/STEP3-2026-Oct-SIM2-v202612-e9ff4ad7730a.zip.1"
```

Expect status **206**, `Content-Range: bytes 0-99/604902`, and a 100-byte file.
A 200 response does not confirm resume support. A 404 usually means a missing
folder, wrong case, or the wrong base URL.

Local catalog queries, checksums, and byte-for-byte reconstruction of the
original bank passed. This package is not deployed yet. Live host behavior
and installation/reading on an Android device remain to be tested.

## Official hosting documentation (checked 25 October 2026)

- [GitHub Pages setup](https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-github-pages-site)
- [GitHub Pages publishing source](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site)
- [R2 setup](https://developers.cloudflare.com/r2/get-started/)
- [R2 uploads and folders](https://developers.cloudflare.com/r2/objects/upload-objects/)
- [R2 public development URL](https://developers.cloudflare.com/r2/buckets/public-buckets/)
- [R2 object-read range support](https://developers.cloudflare.com/r2/api/s3/api/)
- [Cloudflare Pages range behavior](https://developers.cloudflare.com/pages/configuration/serving-pages/)
