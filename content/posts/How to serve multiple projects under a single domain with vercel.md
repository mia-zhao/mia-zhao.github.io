+++
title = 'How to Serve Multiple Projects Under a Single Domain with Vercel'
date = 2024-07-02T13:02:47-07:00
+++

It's not uncommon to need to serve multiple projects under a single domain. For example, you might have a project for https://your-domain.com, and another for https://your-domain.com/blog. In Next.js, the keyword for such a setup is [**Multi Zone**](https://nextjs.org/docs/pages/building-your-application/deploying/multi-zones).

## Set Up Multi Zones

While the examples from [**with zones**](https://github.com/vercel/next.js/tree/canary/examples/with-zones) cover basic configurations, they often don't address more complex scenarios, such as ensuring static assets work seamlessly across different zones.

First, let's consider how the two projects (**home** and **blog**) work independently. When you start the development server for each project, you might have:

- **home** project running at **localhost:3000**

- **blog** project running at **localhost:3001**

To serve the **blog** at **localhost:3000/blog**, we need to configure the [**next.config.js**](https://nextjs.org/docs/pages/api-reference/next-config-js) files for both projects.

### Configuring the Blog Project

In the blog project, set the **basePath** to "/blog" in [**next.config.js**](https://nextjs.org/docs/pages/api-reference/next-config-js).

```
/** @type {import('next').NextConfig} */
const nextConfig = {
  basePath: "/blog",
}

module.exports = nextConfig
```

This configuration adds a path prefix to the **blog** application, so it is now served at **localhost:3001/blog**.

### Configuring the Home Project

To point **localhost:3000/blog** to **localhost:3001/blog**, configure rewrites in the **home** project as follows

```
/** @type {import('next').NextConfig} */
const nextConfig = {
  async rewrites() {
    return [
      {
        source: "/blog/:path*",
        destination: "http://localhost:3001/blog/:path*",
      }
    ];
  }
}

module.exports = nextConfig
```

With the configuration, you can now access **blog** at **localhost:3000/blog**.

## Additional Configurations

After setting up the basic routing, you might notice that the static assets from the blog **blog**, such as CSS and images, are not resolved correctly. To fix this, addition configurations are needed.

### Resolving Static Assets with assetPrefix

Add **assetPrefix** to the **blog** project:

```
/** @type {import('next').NextConfig} */
const nextConfig = {
  basePath: "/blog",
  assetPrefix: "/blog",
}

module.exports = nextConfig
```

In the **home** project, add the following **beforeFiles** to handle static assets:

```
const nextConfig = {
  async rewrites() {
    return {
        beforeFiles: [
            {
                source: "/blog/_next/:path*",
                destination: "http://localhost:3001/blog/_next/:path*",
            },
            {
                source: "/blog/assets/:path*",
                destination: "http://localhost:3001/blog/assets/:path*",
            },
        ],
        fallback: [
            {
                source: "/blog/:path*",
                destination: "http://localhost:3001/blog/:path*",
            },
        ],
    }
  }
}

module.exports = nextConfig
```

### Handling Trailing Slashes

By default, **trailingSlash** is undefined in Next.js. In this case, visiting a path with or without a trailing slash will not redirect, which is not recommended for SEO. To enforce a consistent URL structure, you can set **trailingSlash** to true or false in [**next.config.js**](https://nextjs.org/docs/pages/api-reference/next-config-js).

Setting up Multi Zones in Next.js allows you to serve multiple projects under a single domain efficiently. By configuring **basePath**, **rewrites**, **assetPrefix**, and **trailingSlash**, you can ensure a seamless and consistent user experience.
