---
title: "Setting up hugo blog with github pages and namecheap"
date: 2022-08-01T09:14:39+05:00
draft: false
---

This is first post on the blog where we will see how to we can setup our blog using hugo, github pages and namecheap avoiding any hosting costs.

### At Hosting/Namecheap
- buy domain example.com from namecheap
- point example.com to www.example.com
point www to username.github.io
Add multiple A records which points to github IPs
```
    CNAME @ www.example.com
    CNAME www username.github.io
    A   @   185.199.108.153
    A   @   185.199.109.153
    A   @   185.199.110.153
    A   @   185.199.111.153

```
### On github side.
- create repository username.github.io
- goto that repo settings>pages
- Change folder to docs instead of root and hit save

In custom domain chekbox enter domain www.example.com and hit save
also check Enforce HTTPS.

Inside docs folder just create test index.html file with some hello content.

Next install hugo cli
```bash
hugo new site example.com
cd example.com
git init
git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke

echo theme = \"ananke\" >> config.toml

hugo new posts/my-first-post.md

```
Enter below lines as post header and add some content.

```
---
title: "My First Post"
date: 2019-03-26T08:47:11+01:00
draft: true
---
Testing 123

```
Run hugo command to test the server
`hugo server -D`

Update config.toml with required title and domain to `https://www.example.com`

also change publishdir to 'docs' because that folder will be used by github for static content hosting


```javascript
console.log("ok")
```
