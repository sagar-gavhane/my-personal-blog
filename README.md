# Building a blog with Next.js

Hi there, I hope you're doing well. 😊

My name is Sagar, and I'm working as a software engineer at [Fabric](https://fabric.inc/). I would like to share my thoughts and experiences. Building a blazing-fast blog is my dream project and many times I started but I failed in the middle. After the release of Next.js's 9.3 we can easily generate static pages with this help of [SSG](https://nextjs.org/blog/next-9-3#next-gen-static-site-generation-ssg-support) (Static Site Generation) APIs. In this blog post, we're going to build a blog website from scratch using [Next.js](https://nextjs.org/). Before we start writing code I want to answer one question.

### **In this blog post**

- Why I choose Next.js over Gatsby?
- Project setup
- App structure
- Create blog content
- What is `getStaticProps()` method?
- What is `getStaticPaths()` method?
- Conclusion
- References

### **Why I choose Next.js over Gatsby?**

Here, I don't want to say [Gatsby](https://www.gatsbyjs.org/) is bad. Next.js and Gatsby have their own advantages. But I found that with Gatsby I've to do extra configuration and with Next.js we don't need it. And also there are many Gatsby plugin available to us to easy our development pain. 

There is a good article available to compare Next.js and Gatsy features.

1. [https://www.gatsbyjs.org/features/jamstack/gatsby-vs-nextjs](https://www.gatsbyjs.org/features/jamstack/gatsby-vs-nextjs)
2. [https://blog.logrocket.com/next-js-vs-gatsbyjs-a-developers-perspective/](https://blog.logrocket.com/next-js-vs-gatsbyjs-a-developers-perspective/)
3. [https://dev.to/jameesy/gatsby-vs-next-js-what-why-and-when-4al5](https://dev.to/jameesy/gatsby-vs-next-js-what-why-and-when-4al5)

Enough theory let's start coding…

### **Project setup**

Create the project folder and initialize it using npm.

```bash
mkdir my-personal-blog 
cd my-personal-blog
npm init --y
```

`npm init --y` command will create `package.json` file at root level.

Install `next`, `react`, and `react-dom` in your project. Make sure you're `next.js` version is 9.3 or later else SSG APIs will not work.

```bash
npm install next react react-dom --save
npm install uuid unified remark-html remark-highlight.js remark-parse gray-matter --save-dev
```

Okay, wait for a while let me quickly explain project dependencies.

1. **uuid** - For the creation of RFC4122 UUIDs.
2. **unified** - Interface for parsing, inspecting, transforming, and serializing content through syntax trees.
3. **remark-html** - Remark plugin to compile Markdown to HTML
4. **remark-highlight.js** -  Remark plugin to highlight code blocks with highlight.js.
5. **remark-parse** - Remark plugin to parse Markdown
6. **gray-matter** - Parse front-matter from a string or file.

Open `package.json` and add the following scripts:

```json
"scripts": {   
  "dev": "next",   
  "build": "next build",
  "start": "next start"
}
```

### **App structure**

Before we start writing code, go ahead and set up your folder structure so it looks like this:

```
/my-personal-blog/
|--/components
|--/node_modules
|--/contents
|--/pages
|----/index.js
|----/blog
|------/[slug].js
|--/styles
|----/global.css
|--/utils
|--package.json
```

### **Create blog content**

One more step, Let's add a `hello-world.md` file to our project's `contents` folder, create a file with name `hello-world.md`, and add below markdown content. Later, we'll render this content on the website.

```markdown
---
title: My first blog
slug: hello-world
date: "31-05-2020"
---

Pellentesque condimentum velit vel justo rutrum, sit amet commodo diam tincidunt. Nunc diam massa, interdum ut aliquet at, scelerisque ac ex. Integer cursus sem ac pretium posuere. Ut at odio nulla. Phasellus nec ante luctus, egestas dui id, maximus dui. In aliquam elit sit amet sollicitudin luctus. Nunc nec leo quis ante vestibulum egestas. In dignissim libero vitae congue bibendum. Sed iaculis eros a leo pellentesque, et ultrices leo malesuada. Nullam ultrices rutrum accumsan. Pellentesque tempus sapien et vestibulum placerat.

Donec ultrices in tortor eget facilisis. Pellentesque orci risus, vulputate consequat fermentum eget, euismod sed nulla. Sed luctus sapien quis magna lobortis porttitor. In porttitor nibh id tincidunt imperdiet. Suspendisse ultricies tellus dolor, et gravida tortor vehicula quis. Maecenas tempus est sit amet congue rhoncus. Vivamus vitae felis lacinia, viverra nibh id, pulvinar eros. In viverra venenatis ligula, vitae efficitur felis vehicula vitae. Vestibulum feugiat vel risus iaculis tincidunt.
```

Create a pages directory inside your project and populate `pages/index.js` with the following contents:

```jsx
import React from "react";
import Link from "next/link";

function IndexPage(props) {
  return (
    <div>
      <h1>Blog list</h1>
      <ul>
        {props.blogs.map((blog, idx) => {
          return (
            <li key={blog.id}>
              <Link href={`/blog/${blog.slug}`}>
                <a>{blog.title}</a>
              </Link>
            </li>
          );
        })}
      </ul>
    </div>
  );
}

// This function gets called at build time on server-side.
export async function getStaticProps() {
  const fs = require("fs");
  const matter = require("gray-matter");
  const { v4: uuid } = require("uuid");

  const files = fs.readdirSync(`${process.cwd()}/contents`, "utf-8");

  const blogs = files
    .filter((fn) => fn.endsWith(".md"))
    .map((fn) => {
      const path = `${process.cwd()}/contents/${fn}`;
      const rawContent = fs.readFileSync(path, {
        encoding: "utf-8",
      });
      const { data } = matter(rawContent);

      return { ...data, id: uuid() };
    });

	// By returning { props: blogs }, the IndexPage component
  // will receive `blogs` as a prop at build time
  return {
    props: { blogs },
  };
}

export default IndexPage;
```

Lots of things going on in the above `index.jsx` file. Here, we've created a functional component called `IndexPage` and it's will accepting blogs data as a prop from the `getStaticProps` method. Before understanding code write inside `getStaticProps()` method I would like to explain `getStaticProps()`.

{% twitter 1237096480638451713 %}

### **What is `getStaticProps()` method?**

In simple terms this method **only runs at build-time and will pass props to page component for pre-rendering** and also it doesn't receive any request time data like query parameters or HTTP headers.

Mostly useful in fetching data at build time and the source could be APIs, static files, or even do database queries.

From a performance point of view, If you pre-build pages then there is no need to pass extra bundled js to users. That will drastically increase page interaction time.

---

Let’s back to the `IndexPage` component, if you walk through code written inside `getStaticProps()` you’ll see I’m requiring built-in `fs` module to read a `_content` folder from current directory using `process.cwd()`. `fs.readdirSync(path)` will give me all files listed in `_content` folder. so I’m filtering markdown files only (files that end with .md).

I’m iterating on `files` and passing these file content to the `gray-matter` which will parse front-matter markdown file and return me object which will have `data` and `content` property. In this `getStaticProps()` method we don’t need content so I’m skipping it but on the specific blog page we need it. 

By returning `{ props: blogs }`, the IndexPage component will receive `blogs` as a `prop` at build time.

From `IndexPage` component, I'm mapping over blogs props and rendering all blogs with `Link` tag so we'll able to navigate to specific blog. 

![https://dev-to-uploads.s3.amazonaws.com/i/m4xdjk6sjvnanaxxzzmy.png](https://dev-to-uploads.s3.amazonaws.com/i/m4xdjk6sjvnanaxxzzmy.png)

Now, it's time to accept slug from the query parameter and render blog content onto the screen. Let's create a file called `[slug].js` inside `pages/blog/` folder and take a look at below `BlogPostPage` component. To statically generate all blog posts based on the markdown files, we'll need to specify what path we should generate for. To do this, we need to export an async function `getStaticPaths()`.

```jsx
// file: pages/blog/[slug].js
import React from "react";

function BlogPostPage(props) {
  return (
    <div>
			<h1>{props.blog.title}</h1>
      <section dangerouslySetInnerHTML={{ __html: props.blog.content }}></section>
    </div>
  );
}

// pass props to BlogPostPage component
export async function getStaticProps(context) {
  const fs = require("fs");
  const html = require("remark-html");
  const highlight = require("remark-highlight.js");
  const unified = require("unified");
  const markdown = require("remark-parse");
  const matter = require("gray-matter");

  const slug = context.params.slug; // get slug from params
  const path = `${process.cwd()}/contents/${slug}.md`;
  
	// read file content and store into rawContent variable
	const rawContent = fs.readFileSync(path, {
    encoding: "utf-8",
  });

  const { data, content } = matter(rawContent); // pass rawContent to gray-matter to get data and content

  const result = await unified()
    .use(markdown)
    .use(highlight) // highlight code block
    .use(html)
    .process(content); // pass content to process

  return {
    props: {
			blog: {
				...data,
	      content: result.toString(),
			}
    },
  };
}

// generate HTML paths at build time
export async function getStaticPaths(context) {
  const fs = require("fs");

	const path = `${process.cwd()}/contents`;
  const files = fs.readdirSync(path, "utf-8");
  
	const markdownFileNames = files
    .filter((fn) => fn.endsWith(".md"))
    .map((fn) => fn.replace(".md", ""));

  return {
    paths: markdownFileNames.map((fileName) => {
      return {
        params: {
          slug: fileName,
        },
      };
    }),
    fallback: false,
  };
}

export default BlogPostPage;
```

### **What is `getStaticPaths()` method?**

This method defines a list of paths that have to be rendered to HTML at build time useful if a page has dynamic routes like `blog/[slug].js`.  `Next.js` will statically pre-render all the paths specified by `getStticPaths()`.  From `getStaticPaths()` method mandatory to returns `path` and a `fallback` key. If `fallback` is `false`, then any paths not returned by `getStaticPaths()`  at build time will result in a 404 page.

![https://dev-to-uploads.s3.amazonaws.com/i/u2vx9ycjuvl2x8x8lqea.png](https://dev-to-uploads.s3.amazonaws.com/i/u2vx9ycjuvl2x8x8lqea.png)

Here you'll find git repository: [https://github.com/sagar-gavhane/my-personal-blog](https://github.com/sagar-gavhane/my-personal-blog) 

{% codesandbox my-personal-blog-ou15n %}

### **Conclusion**

Creating a blog website with `Next.js` is pretty straightforward. There are only a few steps we've to follow like reading files, and parsing inside `getStaticProps()` and generating pre-rendered paths using `getStaticPaths()` method. I found that lots of folks are trying to utilizing this powerful feature to pre-render static pages. Recently, Next.js announced an incremental static regeneration feature in `Next.js`  v9.4 this will help us statically pre-render an infinite number of pages. 

### **Reference Links**

1. [https://nextjs.org/blog](https://nextjs.org/blog)
2. [https://github.com/vercel/next.js/tree/canary/examples/blog-starter](https://github.com/vercel/next.js/tree/canary/examples/blog-starter)
3. [https://github.com/tscanlin/next-blog](https://github.com/tscanlin/next-blog)
4. [https://www.gatsbyjs.org/features/jamstack/gatsby-vs-nextjs](https://www.gatsbyjs.org/features/jamstack/gatsby-vs-nextjs)
5. [https://blog.logrocket.com/next-js-vs-gatsbyjs-a-developers-perspective/](https://blog.logrocket.com/next-js-vs-gatsbyjs-a-developers-perspective/)
6. [https://dev.to/jameesy/gatsby-vs-next-js-what-why-and-when-4al5](https://dev.to/jameesy/gatsby-vs-next-js-what-why-and-when-4al5)

![https://i.gifer.com/5qJ.gif](https://i.gifer.com/5qJ.gif)

Thanks for reading. I hope you like this article feel free to like, comment, or share this article with your friends. For a more depth understanding of `Next.js` API's checkout official documentation website.
