---
layout: post
title: How to Create a Post in Jekyll
date: 2024-11-10 15:09:00 +-0000
categories: [Reminders]
tags: [test,jekyll,markdown]
---

## File Formating

A file used to create a post must be named "_YYYY-MM-DD-nameofpost.md_."
That file then must reside within the "__posts_" folder.

Each file used for a post must begin with the front matter as shown below:

```
---
layout: post
title: Title for your post
date: YYYY-MM-DD HH:MM:SS +-0000 #Time denoted using the 24 hour clock "+-0000" used to offset time, from the timezone set in "_config.yml"
categories: [cat1, CAT2, cat3] #Categories must be separated by a comma
tags: [tag1,tag2,tag3] #Tags function like an array, must be separated by a comma, lowercase, no excess whitespace
---
```

## References

Below are the links used to build both this post and the previous Hello Homelab post.

Sites and Docs

[Chirpy Theme Writing a New Post](https://chirpy.cotes.page/posts/write-a-new-post/#table-of-contents)

[Chirpy Theme Text and Typography](https://chirpy.cotes.page/posts/text-and-typography/)

[Techno Tim Documentation](https://technotim.live/posts/jekyll-docs-site/)

[Techno Tim Video](https://www.youtube.com/watch?v=F8iOU1ci19Q)

[Techno Tim Repo](https://github.com/techno-tim/techno-tim.github.io)

[Jekyll Documentation](https://jekyllrb.com/)

[Jekyll Documentation Posts](https://jekyllrb.com/docs/posts/)

[Jekyll Documentation Tags](https://jekyllrb.com/docs/liquid/tags/#linking-to-posts)
