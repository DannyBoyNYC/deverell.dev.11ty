---
title: Dev Notes
date: "2020-05-03"
description: "Notes about maintaining and updating this Gatsby site."
---

Installed `"gatsby-background-image": "^1.1.1",`.

Resolved image processing failures with:

```js
{
  resolve: `gatsby-transformer-remark`,
  options: {
    plugins: [
      {
        resolve: `gatsby-remark-images`,
        options: {
          maxWidth: 800,
        },
      },
    ],
  },
},
```

Images are referenced within the post as `![Conrad](./conrad-johnson-preamp.png) `

 Discovered:

 `resolve: 'gatsby-plugin-manifest',` - a tool for managing `.ico` files

 Added:

 `hero.js` - an image at the top using `import BackgroundImage from "gatsby-background-image"`