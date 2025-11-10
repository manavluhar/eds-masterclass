# Advanced Topics Hands-on Exercise

This is a hands-on exercise. You will be migrating some content from https://deno.com over to Edge Delivery on DA, doing some light coding and updating some configurations via the Admin API.

## Prerequisites
As a pre-requisite to Masterclass, you should already have an org and a site up and running after having gone through the [Getting Started tutorial](https://www.aem.live/developer/tutorial). Lets expand on that now to:
- Create a new site under your org for Deno
- Import some blog content into the site
- Create two separate indices & sitemaps for the blog and site pages
- Create a repoless site called _deno-enterprise_
- Add a custom header for some path(s)
- Add a new user to the org

**IMPORTANT: Before going through the following steps, fork this `eds-masterclass` repo.**

## References:

- Admin API Docs: https://www.hlx.live/docs/admin.html#tag/code/oper
- aem.live docs: https://www.aem.live/docs/

## Step 1: Create the Deno site in your org

```sh
curl -X POST https://admin.hlx.page/config/{org}/sites/deno.json \
  -H 'content-type: application/json' \
  -H 'x-auth-token: {your-auth-token}' \
  --data '{
  "code": {
    "owner": "{org}",
    "repo": "{site}"
  },
  "content": {
    "source": {
      "url": "https://content.da.live/{org}/{site}/"
    }
  }
}'
```
Alternatively:
- Go to https://labs.aem.live/tools/site-admin/index.html (make sure you are authenticated via AEM Sidekick)
- Create a new site
- Point it to your fork of this `eds-masterclass` repo
- Create a folder called `deno` in DA for the site, and point the site's content to https://content.da.live/{ORG}/deno/
- Copy the boilerplate content from your original site created with the DA tutorial to this new site. We just need some sample content.

<img width="1196" height="394" alt="image" src="https://github.com/user-attachments/assets/2951b6a8-7a35-4939-b4e2-bc8e75c82c15" />

## Step 2: Import content
- Run `aem import` from the repo
- Review the transformation file at `/tools/importer/import.js`.
- Import the following pages:
    - https://deno.com/blog/fresh-and-vite
    - https://deno.com/blog/updates-from-tc39
    - https://deno.com/blog/deno-deploy-highlights
    - https://deno.com/blog/open-source
    - https://deno.com/blog/v2.5

TASK: The import currently mostly works fine, but we need to enhance it to:
- Include the `Author` and `Tags` of the post as metadata which are currently missing.
- Remove the author and tags that are being imported as actual content since we're utilizing page metadata for that.

Update the `handleBlogPosts` function in `import.js` to include these in the metadata table. We also want to remove the author and tags from the actual content that is being imported since that is being placed in the metadata.

_Solution is available here: https://gist.github.com/usman-khalid/d15cdec210ff88fd9db55e022a3804fb_

_TIP: Use the Import Workbench to import one as a test. Once satisfied with your import updates, import all 5 pages in bulk, then drop them in your DA folder in a /blog folder._

## Step 3: Create site and blog indices
- Create a `default` index for site content by running the following command. It should include all site content excluding blog post pages:

```sh
curl -X POST https://admin.hlx.page/config/{org}/sites/{site}/content/query.yaml \
  -H 'content-type: text/yaml' \
  -H 'x-auth-token: {your-auth-token}' \
  --data @resources/index-config.yaml
```

- Then, go to https://labs.aem.live/tools/index-admin/index.html to review your index config.
- Add a second index called `blog` using the index admin tool.
    - Value for `include` should be `/blog/**`  

## Step 4: Create sitemaps pointing to their indices
- Go to https://labs.aem.live/tools/sitemap-admin/index.html
- Create sitemap configs for blog and site content pointing to the indices created in Step 3
- Generate the sitemaps for both
- Verify that things look correct at `/sitemap.xml` and `/blog-sitemap.xml`

If you prefer to use curl, use the following command:

```sh
curl -X POST https://admin.hlx.page/config/{org}/sites/{site}/content/sitemap.yaml \
  -H 'content-type: text/yaml' \
  -H 'x-auth-token: {your-auth-token}' \
  --data @resources/sitemap-config.yaml
```

## Step 5: Create a repoless site

We now want a second site to utilize the same code & block library.

- Create a second "repoless" site, pointing to the same codebase
- Create a metadata sheet for the new site and apply the `enterprise` theme to all pages.
    - The key/value pair should be `URL` and `theme`
    - We can apply it to all paths on the site using `/**` 
- Review how themes are loaded in `scripts/aem.js`
- Change the value of the `--link-color` CSS variable scoped to `body.enterprise`
- Test your changes on both sites!

<img width="1339" height="462" alt="image" src="https://github.com/user-attachments/assets/cb051888-0fb1-45f0-9e0b-59263db1ebd1" />

You now have two sites with a separatation in styling powered by the same codebase!

_TIP: You can run dev server for both sites locally by running `aem up` with the `--pagesUrl` and `--port` flags pointing to the their respective URLs._

Repoless Docs: https://www.aem.live/docs/repoless

## Step 6: Add a custom header
- Apply a custom HTTP header to any content or code path. You can use Postman, Curl or the HTTP Headers Editor tool: https://labs.aem.live/tools/headers-edit/index.html

CURL:
```sh
curl -X POST https://admin.hlx.page/config/{org}/sites/{site}/headers.json \
  -H 'content-type: application/json' \
  -H 'x-auth-token: {your-auth-token}' \
  --data '{
	"/**": [
      {
        "key": "access-control-allow-origin",
        "value": "*"
      }
    ]
}'
```
- Verify that the response header exists by making a `GET` request to the resource

## Step 7: Add a new user to your org
- Go to https://labs.aem.live/tools/user-admin/index.html
- Add a new user to your org with the role `config_admin`

## Step 8: Add a redirect
- Create a sheet in the content root called `redirects`
- Add 2 columns: `source` and `destination`
- Add a redirect for `/news` to redirect to `/blog`
- Test the redirect on the site

_Note: These paths should be relative and not the FQDN_

