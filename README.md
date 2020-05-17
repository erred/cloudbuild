# cloudbuild

A repo for cloudbuild

[![License](https://img.shields.io/github/license/seankhliao/cloudbuild.svg?style=flat-square)](LICENSE)
![Version](https://img.shields.io/github/v/tag/seankhliao/cloudbuild?sort=semver&style=flat-square)

## cloudflare

```yaml
steps:
  - id: cloudflare-cache-flush
    name: curlimages/curl
    entrypoint: /bin/bash
    secretEnv:
      - ZONE
      - TOKEN
    args:
      - -c
      - |
        curl -X POST \
        "https://api.cloudflare.com/client/v4/zones/$$ZONE/purge_cache" \
          -H "Authorization: Bearer $$TOKEN" \
          -H "Content-Type:application/json" \
          -d '{"files":[ \
            "https://seankhliao.com/blog/", \
            "https://seankhliao.com/like/", \
            "https://seankhliao.com/ideas/" \
            ]}'
```

## container / kaniko

```yaml
steps:
  - id: login
    name: gcr.io/cloud-builders/gcloud
    entrypoint: /bin/bash
    args:
      - -c
      - gcloud secrets versions access latest --secret=registries > /login/config.json
    volumes:
      - name: login
        path: /login

  - id: build-push
    name: gcr.io/kaniko-project/executor:latest
    args:
      - -c=.
      - -f=Dockerfile
      - -d=$_REG/$PROJECT_ID/$_IMG:latest
      - -d=$_REG/$PROJECT_ID/$_IMG:$TAG_NAME
      - -d=index.docker.io/seankhliao/$_IMG:latest
      - -d=index.docker.io/seankhliao/$_IMG:$TAG_NAME
      - --reproducible
      - --single-snapshot
    volumes:
      - name: login
        path: /kaniko/.docker
```

## git

```yaml
- id: git-push
  name: gcr.io/cloud-builders/git
  entrypoint: /bin/sh
  args:
    - -c
    - |
      set -x &&
      git config user.email ... && \
      git config user.name ... && \
      git remote set-url origin https://${GITHUB_TOKEN}:x-oauth-basic@github.com/... && \
      git fetch origin a_branch && \
      git checkout a_branch && \
      git add * && \
      git commit -m "..." && \
      git push origin a_branch
```

## web archive

### inline

```yaml
steps:
  - id: add-to-web-archive
    name: curlimages/curl
    entrypoint: /bin/bash
    args:
      - -c
      - |
        cat << EOF | xargs -I '{}' -n 1 curl -Lv https://web.archive.org/save/'{}'
        // url1
        // url2
        EOF
```

### file

```yaml
steps:
  - id: add-to-web-archive
    name: curlimages/curl
    entrypoint: /bin/bash
    args:
      - -c
      - |
        xargs -I '{}' -n 1 curl -Lv https://web.archive.org/save/'{}' < sitemap.txt
```
