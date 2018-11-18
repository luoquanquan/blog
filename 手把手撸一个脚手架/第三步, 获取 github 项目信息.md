系列文章:

- [【手把手带你撸一个脚手架】第一步, 创建第一个命令](https://juejin.im/post/5bead1b25188251e1a1f4d34)
- [【手把手带你撸一个脚手架】第二步, 搭建开发环境](https://juejin.im/post/5bec24ddf265da61171c4a34)
- [【手把手带你撸一个脚手架】第三步, 获取 github 项目信息](https://juejin.im/post/5bec598d51882579117f61f8)
- [【手把手带你撸一个脚手架】第四步, 通过撸码获取项目信息](https://juejin.im/post/5bed6ff2f265da61137ed948)
- [【手把手带你撸一个脚手架】第五步, 撸完收工](https://juejin.im/post/5beed37b51882527796a9d8e)

> 脚手架, 作为一个工具, 主要是用来搬运我们已经准备好的项目模板(webpack 配置, koa 项目雏形等等), 这一步我们就来创建用于搬运的项目模板. 并通过接口获取相关信息 [github Api 文档](https://developer.github.com/v3/repos/)

ps: 这一步基于 github 若有没使用过的小伙伴, 请加油

![2018-11-15-00-33-46](https://user-gold-cdn.xitu.io/2018/11/15/167133d84ce3d1b0?w=692&h=586&f=png&s=186981)

## 创建一个专门用于维护项目模板的项目组

> 为了不和自己平时写的各种辣鸡代码混杂在一起, 这里我专门创建了一个 organization 不会创建的兄弟们请 [度娘](https://www.baidu.com/s?ie=UTF-8&wd=github%20%E5%88%9B%E5%BB%BA%20organization)

- 目录切换到刚刚创建的 organization 上

![2018-11-15-00-38-51](https://user-gold-cdn.xitu.io/2018/11/15/167133d84cce5583?w=2114&h=844&f=png&s=295519)

- 创建一个项目

![2018-11-15-00-39-18](https://user-gold-cdn.xitu.io/2018/11/15/167133d84cdc2b6c?w=2004&h=1034&f=png&s=332244)

- 向创建的项目中添加一个文件, 上传到 github 并打好 tag

![2018-11-15-00-44-17](https://user-gold-cdn.xitu.io/2018/11/15/167133d86f7dc7dc?w=1994&h=892&f=png&s=206731)

准备工作完成 ^_^

![2018-11-15-00-49-08](https://user-gold-cdn.xitu.io/2018/11/15/167133d896663b93?w=410&h=548&f=png&s=334554)

## 通过 github 开放 api 获取项目信息

> baseUrl: [https://api.github.com](https://api.github.com)

作为一个开发工具, 我们需要获取的项目信息包含以下几个:

### 获取组织所属项目列表[文档](https://developer.github.com/v3/repos/#list-organization-repositories)

我们可以尝试一下获取刚刚创建的 organization 下的项目目录

```js
curl https://api.github.com/orgs/learn-cli-organization/repos

// 返回的结果
[
  {
    "id": 157579674,
    "node_id": "MDEwOlJlcG9zaXRvcnkxNTc1Nzk2NzQ=",
    "name": "demo",
    "full_name": "learn-cli-organization/demo",
    "private": false,
    "owner": {
      "login": "learn-cli-organization",
      "id": 45043923,
      "node_id": "MDEyOk9yZ2FuaXphdGlvbjQ1MDQzOTIz",
      "avatar_url": "https://avatars2.githubusercontent.com/u/45043923?v=4",
      "gravatar_id": "",
      "url": "https://api.github.com/users/learn-cli-organization",
      "html_url": "https://github.com/learn-cli-organization",
      "followers_url": "https://api.github.com/users/learn-cli-organization/followers",
      "following_url": "https://api.github.com/users/learn-cli-organization/following{/other_user}",
      "gists_url": "https://api.github.com/users/learn-cli-organization/gists{/gist_id}",
      "starred_url": "https://api.github.com/users/learn-cli-organization/starred{/owner}{/repo}",
      "subscriptions_url": "https://api.github.com/users/learn-cli-organization/subscriptions",
      "organizations_url": "https://api.github.com/users/learn-cli-organization/orgs",
      "repos_url": "https://api.github.com/users/learn-cli-organization/repos",
      "events_url": "https://api.github.com/users/learn-cli-organization/events{/privacy}",
      "received_events_url": "https://api.github.com/users/learn-cli-organization/received_events",
      "type": "Organization",
      "site_admin": false
    },
    "html_url": "https://github.com/learn-cli-organization/demo",
    "description": null,
    "fork": false,
    "url": "https://api.github.com/repos/learn-cli-organization/demo",
    "forks_url": "https://api.github.com/repos/learn-cli-organization/demo/forks",
    "keys_url": "https://api.github.com/repos/learn-cli-organization/demo/keys{/key_id}",
    "collaborators_url": "https://api.github.com/repos/learn-cli-organization/demo/collaborators{/collaborator}",
    "teams_url": "https://api.github.com/repos/learn-cli-organization/demo/teams",
    "hooks_url": "https://api.github.com/repos/learn-cli-organization/demo/hooks",
    "issue_events_url": "https://api.github.com/repos/learn-cli-organization/demo/issues/events{/number}",
    "events_url": "https://api.github.com/repos/learn-cli-organization/demo/events",
    "assignees_url": "https://api.github.com/repos/learn-cli-organization/demo/assignees{/user}",
    "branches_url": "https://api.github.com/repos/learn-cli-organization/demo/branches{/branch}",
    "tags_url": "https://api.github.com/repos/learn-cli-organization/demo/tags",
    "blobs_url": "https://api.github.com/repos/learn-cli-organization/demo/git/blobs{/sha}",
    "git_tags_url": "https://api.github.com/repos/learn-cli-organization/demo/git/tags{/sha}",
    "git_refs_url": "https://api.github.com/repos/learn-cli-organization/demo/git/refs{/sha}",
    "trees_url": "https://api.github.com/repos/learn-cli-organization/demo/git/trees{/sha}",
    "statuses_url": "https://api.github.com/repos/learn-cli-organization/demo/statuses/{sha}",
    "languages_url": "https://api.github.com/repos/learn-cli-organization/demo/languages",
    "stargazers_url": "https://api.github.com/repos/learn-cli-organization/demo/stargazers",
    "contributors_url": "https://api.github.com/repos/learn-cli-organization/demo/contributors",
    "subscribers_url": "https://api.github.com/repos/learn-cli-organization/demo/subscribers",
    "subscription_url": "https://api.github.com/repos/learn-cli-organization/demo/subscription",
    "commits_url": "https://api.github.com/repos/learn-cli-organization/demo/commits{/sha}",
    "git_commits_url": "https://api.github.com/repos/learn-cli-organization/demo/git/commits{/sha}",
    "comments_url": "https://api.github.com/repos/learn-cli-organization/demo/comments{/number}",
    "issue_comment_url": "https://api.github.com/repos/learn-cli-organization/demo/issues/comments{/number}",
    "contents_url": "https://api.github.com/repos/learn-cli-organization/demo/contents/{+path}",
    "compare_url": "https://api.github.com/repos/learn-cli-organization/demo/compare/{base}...{head}",
    "merges_url": "https://api.github.com/repos/learn-cli-organization/demo/merges",
    "archive_url": "https://api.github.com/repos/learn-cli-organization/demo/{archive_format}{/ref}",
    "downloads_url": "https://api.github.com/repos/learn-cli-organization/demo/downloads",
    "issues_url": "https://api.github.com/repos/learn-cli-organization/demo/issues{/number}",
    "pulls_url": "https://api.github.com/repos/learn-cli-organization/demo/pulls{/number}",
    "milestones_url": "https://api.github.com/repos/learn-cli-organization/demo/milestones{/number}",
    "notifications_url": "https://api.github.com/repos/learn-cli-organization/demo/notifications{?since,all,participating}",
    "labels_url": "https://api.github.com/repos/learn-cli-organization/demo/labels{/name}",
    "releases_url": "https://api.github.com/repos/learn-cli-organization/demo/releases{/id}",
    "deployments_url": "https://api.github.com/repos/learn-cli-organization/demo/deployments",
    "created_at": "2018-11-14T16:41:01Z",
    "updated_at": "2018-11-14T16:42:39Z",
    "pushed_at": "2018-11-14T16:43:18Z",
    "git_url": "git://github.com/learn-cli-organization/demo.git",
    "ssh_url": "git@github.com:learn-cli-organization/demo.git",
    "clone_url": "https://github.com/learn-cli-organization/demo.git",
    "svn_url": "https://github.com/learn-cli-organization/demo",
    "homepage": null,
    "size": 0,
    "stargazers_count": 0,
    "watchers_count": 0,
    "language": "JavaScript",
    "has_issues": true,
    "has_projects": true,
    "has_downloads": true,
    "has_wiki": true,
    "has_pages": false,
    "forks_count": 0,
    "mirror_url": null,
    "archived": false,
    "open_issues_count": 0,
    "license": null,
    "forks": 0,
    "open_issues": 0,
    "watchers": 0,
    "default_branch": "master",
    "permissions": {
      "admin": false,
      "push": false,
      "pull": true
    }
  }
]
```

拿到这个数组说明我们已经能够获取到项目组中的所有项目啦, 一波猝不及防的商业互吹 ^_^

![16705bca01c5e44c](https://user-gold-cdn.xitu.io/2018/11/15/167153edcfeb3654?w=150&h=89&f=gif&s=64454)

### 获取指定项目的版本号 [文档](https://developer.github.com/v3/repos/#list-tags)

通过前一个接口, 我们成功的获取到了项目组中所有的项目信息, 接下来我们可以通过以下接口获取到指定项目的版本信息(就是 tags)

```js
curl https://api.github.com/repos/learn-cli-organization/demo/tags

// 返回结果
[
  {
    "name": "v0.0.1",
    "zipball_url": "https://api.github.com/repos/learn-cli-organization/demo/zipball/v0.0.1",
    "tarball_url": "https://api.github.com/repos/learn-cli-organization/demo/tarball/v0.0.1",
    "commit": {
      "sha": "00f0dda86e5f922e2ae406c25e19b44b2463f690",
      "url": "https://api.github.com/repos/learn-cli-organization/demo/commits/00f0dda86e5f922e2ae406c25e19b44b2463f690"
    },
    "node_id": "MDM6UmVmMTU3NTc5Njc0OnYwLjAuMQ=="
  }
]
```

下集预告: 到目前为止, 我们已经能够获取到项目信息. 下一步我们会将结合 `inquirer.js` 实现命令行交互式的动态获取这些信息

![2018-11-15-01-19-33](https://user-gold-cdn.xitu.io/2018/11/15/167133d89d588be0?w=660&h=552&f=png&s=373369)
