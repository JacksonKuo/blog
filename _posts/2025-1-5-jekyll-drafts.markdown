---
layout: post
title: "Better Jekyll Drafts"
date: 2025-1-5
tags: ["jekyll"]
published: false
---

**Contents**
* TOC
{:toc}

# Problem Statement

Come up with an alterative method for Jekyll drafts, since the offical draft process in Jekyll kinda sucks. The following is how the official draft method works:

* Since `drafts are posts without a date in the filename`[^1], publishing requires adding a date to the filename.
* Publishing requires moving the post file from `_drafts` to the `_posts` folder
* To view drafts, the `--drafts` switch is required

Modifying dates in the filename and moving the file in order to publish is pretty annoying. Ideally the an author should only need to modify the front matter to publish

# Solution

A better method is to save the draft in the `_posts` folder, but to do the following:

1. Set the `--unpublished`switch:
    * `bundle exec jekyll serve --unpublished`
2. To create a local post, but not display the post
    * `published: false`
    * set the `date: 2025-xx-xx` to a day far in the future. Jekyll will not display posts with a future date. This way you can create as many local drafts as you want, without making a mess of the local blog
3. To toggle displaying the draft locally, change the date to the current day.
4. To publish the draft in Github Pages, change `published` to true

# References

[^1]: [https://jekyllrb.com/docs/posts/](https://jekyllrb.com/docs/posts/)