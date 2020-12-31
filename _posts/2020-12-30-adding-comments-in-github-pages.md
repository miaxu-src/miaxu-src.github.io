---
category: jekyll
---

It would be awesome if someone can leave comments to your posts. However, because a GitHub page is a static website, allowing to leave a comment is not a typical use case. After a few days online research, I found a very useful 3rd-party GitHub App called <a href="https://utteranc.es/">Utterances</a> that can achieve this goal.

According to the Utterances' website, it uses GitHub issues for blog comments. The installation is also straighforward:
1. Log into your GitHub account.
2. Go to <a href="https://github.com/apps/utterances">this page</a> and choose to install this GitHub App.
3. During the installation, choose the repo that you want to enable the comments feature.
4. Create an html, e.g., comments.html,  in your `_includes` directory.
5. To enable Utterances, add a script tag to the comments.html. Mine looks like this:
```html
<script src="https://utteranc.es/client.js"
		repo="miaxu-src/miaxu-src.github.io"
		issue-term="pathname"
		theme="github-light"
		crossorigin="anonymous"
		async>
</script>
```
6. In your post layout page, include the comments.html.

That's it! You will have a nice-looking comment system on your post page.
