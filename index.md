# Welcome!
Welcome to Blocky Bentley, the home of Blocky's experiments, written as blogs

---

# Writing Your First Blog

---

## Author Listing
You will notice that there is an [Author page](https://ianhecker.github.io/test-website/authors.html). If you are not listed as an author, contact the site's maintainer so that you can write posts

## Writing a Blog
Writing blogs is easy! Follow the below steps

---

### Clone the Github Repository
```
git clone {{ site.github.repository_url }}.git
```
Or
```
git clone git@github.com:blocky/{{ site.github.repository_name }}.git
```

### Markdown File
Now, in the [_posts directory](https://github.com/ianhecker/test-website/tree/main/_posts), we will create a file

Let's name your post
```
POST=$(date +"%Y-%m-%d"-my-awesome-post.md)
```

Now, let's get your first name in lowercase
```
AUTHOR=$(echo "your-first-name" | tr [:upper:] [:lower:])
```

Now, echo some [Front Matter](https://jekyllrb.com/docs/front-matter/) into a file
```
cat <<EOF >_posts/$POST
---
layout: post
author: $AUTHOR
---
EOF
```
By adding your first name, the website will list your posting in your Author page

---
You are ready to create content!

### Content
Now, fill the markdown file with your awesome experiment! Use markdown to format your text

### Publishing
Publishing is as simple as pushing your markdown post to the Github repository's main branch

Then, Github Actions will publish your posting to the [blog page](https://ianhecker.github.io/test-website/blog.html)

You should also be able to see your posts on your author page
