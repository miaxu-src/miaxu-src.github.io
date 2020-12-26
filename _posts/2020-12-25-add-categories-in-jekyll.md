---
category: jekyll
---

I recently added categories, i.e., grouping my blogs by topics, into my website. To group blogs in Jekyll, you will need to use <em>collections</em>. Basically I followed the instructions <a href="https://jekyllrb.com/docs/step-by-step/09-collections/">here</a> to create my blog categories, which is quite straightforward. Here I'm providing a quick summary:


1. In the `_config.yml` file, adding following lines:
    ```yaml
    collections:
      categories:
        output: true
    ```

2. Creating a directory `_categories`. Inside this directory, creating files such as `jekyll.md`. You will need to create a file for each category.

3. In the file, specifying as many properties as you want in the <a href="https://jekyllrb.com/docs/step-by-step/03-front-matter/">front matter</a>. For example,
    ```markdown
    ---
    short_name: jekyll
    name: Jekyll
    ---
    ```

    Later on you will use those defined properties to link you post into a category.

4. Adding a page, e.g., `category.md`, to list all categories.

5. For your post, adding a category property in the front matter. For example, adding `category: jekyll`. The property's value should be one of your categories.

6. The last but not the least, in your navigation bar, adding a tab for the category page so that you can jump into that page.

That's it. For a detailed example, you can find it on Jekyll's website.

