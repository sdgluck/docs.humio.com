[![Build Status](https://drone.humio.cloud/api/badges/humio/docs.humio.com/status.svg)](https://drone.humio.cloud/humio/docs.humio.com)

# docs.humio.com

Official documentation for Humio

## Building docs

First, you need to make sure you have [Hugo installed](https://gohugo.io/getting-started/quick-start/#step-1-install-hugo) on your machine.
Then you can preview the docs by running

```
$ make run
```

which will expose the documentation at [localhost:1313](http://localhost:1313).

## Updating online docs

All changes to the master branch are almost immediately deployed to [docs.humio.com](https://docs.humio.com).

## Permalinks

If you link to a page from an external app (such as Humio's UI) you should use
the permalink for the page. You can create a permalink by adding an alias to the
page's frontmatter. E.g.

```
---
aliases: ["/ref/my-topic"]
---
```

Now no matter where we move that page in the future, your external link will continue
to work.
