---
layout: post
title:  "Building My Brother's Professional Website with Gatsby.js"
date:   2021-02-13 17:00:00 -0500
categories: code
---

I recently built my talented brother a new [professional website](https://liamkaplanmusic.com). Here's what I learned.

### The background

In high school, I built three or four websites using popular drag and drop web technologies like Wordpress and Squarespace. I was always impressed by the speed and possibilities using these tools, but was frustrated with the level of abstraction involved. Often, I'd have a specific idea of how I wanted something to look but couldn't hack the tools available to make it happen. Of course, I was getting **a lot** of things handled for me under the hood in exchange—something which is much clearer to me now than it was then.

### Site philosophy

My brother being an accomplished pianist and composer, I felt that the goal of his website was to present him as an artist with **as little friction as possible**. My guiding principles were "clean, fast, and mobile-optimized." To achieve this, it was important to have consistent design principles across the website (down to spacing colors and layouts) and to use a fast and reactive framework.

### Enter Gatsby.js

After seeing getting into software engineering twitter and stumbling upon [Dan Abramov](https://twitter.com/dan_abramov?ref_src=twsrc%5Egoogle%7Ctwcamp%5Eserp%7Ctwgr%5Eauthor), one of the creators of React, I noticed that his [website](https://overreacted.io/) was built using [Gatsby.js](https://www.gatsbyjs.com/) and wanted to give the technology a try. I am very, very happy with the tool. Gatsby is a React based framework with a number of cool features that are easy to use—server side rendering, caching, lazy image loading, and tightly integrated HTML, CSS, and JS.

### Server side page loading

Gatsby's `<Link>` tag uses SSR to create nearly instant transitions between pages, reducing the friction of navigation and helping your site feel more like an app.

### Lazy image loading

When Gatsby builds your site, it creates many copies of the images you've added to the site in different resolutions. In this way, you can use graphQL to to query many versions of the requested image, so one image doesn't slow down the overall page load. This results in your images appearing fuzzy very briefly and then coming into focus. I ended up querying images like this (of course, there are many many different ways to query graphQL. I took the laziest approach by using the "name" field):

```
const AlbumOne = () => {
  const data = useStaticQuery(graphql`
    query {
      file(name: { eq: "albumOneStatic" }, extension: {eq: "jpg"}) {
        childImageSharp{
          fluid(maxWidth:500,quality:100) {
            ...GatsbyImageSharpFluid
            ...GatsbyImageSharpFluidLimitPresentationSize
          }
        }
      }
    }
  `
  )
```

`...GatsbyImageSharpFluid` is shorthand for a suite of image settings that Gatsby provides out of the box. I've found it to look quite good.

### An example of all the pieces coming together

Gatsby is all about building React components. [React helmet](https://www.npmjs.com/package/react-helmet), a tool built by the NFL is a great example of this.

To maintain a navbar, footer, custom font, and SEO across the site, I used React Helmet like this:

```
export default ({ children }) => (
  <>
    <SEO>
     <link rel="stylesheet" href="https://use.typekit.net/syk0xbq.css"></link>
    </SEO>
    <div>
      <header>
        <h2>
          <Navbar>

          </Navbar>
        </h2>
      </header>
      {children}
      <footer> 
        <p>
        <span style={footerStyle}> {`© ${new Date().getFullYear()} Liam Kaplan`} </span>
        </p>
        <p>
            <span style = {socialGrid}> <Spotify/> <Apple/> <Facebook/> </span>
        </p>
      </footer>
    </div>
  </>
)
```

As you can see, I have SEO and Navbar components that were defined elsewhere easily integrated across my website this way. All I have to do is wrap the JS page files in the `<Layout>` component defined above.

### How Gatsby works

Gatsby abstracts the transition from React to a blazingly fast site ready to be served. Running `gatsby build` runs all the gatsby and npm module code to create a public folder where the content for your site is generated, including a graphQL database, HTML, JS, and CSS. All of this backend code stays on your machine and doesn't need to be pushed to the web server you choose. For the easiest deployment, you can drag and drop the public folder to Netifly. You can also go through a bit more work to serve your site directly from a git repository on push. 

While developing, you can see your site live by running `gatsby develop` and creating a local server. In this way, you can mess around with styling and layout things interactively without touching the site you've pushed live.

### Where to serve

This [website](https://owenmkaplan.com) is deployed on GitHub Pages. Originally, I intended to serve Liam's site there, but after having trouble deploying initally, I instead opted for [Netifly](https://www.netifly.com). I'm really happy with their product. For free, they offer 100 GB of bandwith per month, nearly unlimited builds, and easy HTTPS settup ("Not secure" banners on your site look terrible and are also a security risk, especially if your site has something like a login or form data. It's not great to generate plain text HTTP requests). For myself, deploying an unencrypted site would be embarrasing given that I'm currently working in Wesleyan's [privacy tech lab](https://www.privacytechlab.org/).

### Speed

Gatsby builds [liamkaplanmusic.com](https://liamkaplanmusic.com) in about 30 seconds and Netifly can serve a new version also in about 30 seconds. This means, after the leg work has been put in, I can update something like an image or spelling or adding content in a few minutes. Gatsby also implements caching to improve load times once the site is live.


### Cost

With gatsby being open source, this means that it will cost $12 per year to maintain a Gatsby + Netifly site with a custom domain. This is a great deal in comparison to the $20/month or so that many services charge. Over time, $20/month bills really add up: 20 * 12 * 10 = $2,220. 

In today's tech landscape every company is clamoring to get a $20/month rent on your credit card. Think Adobe, Spotify, Netflix, Amazon, Apple, Playstation, Hulu, and their ever-creeping subscription costs. Think also of the profusion of the subscription model across industries. This makes Netifly's free tier especially attractive. 

I'm confident in the company's longevity given their recent $53 million in additional funding this past [March](https://news.crunchbase.com/news/investors-serve-up-53m-in-series-c-funding-to-web-dev-platform-netlify/). I'am also confident in their free tier given that they directly compete with Microsoft's GitHub Pages.

### Concluding thoughts

Building [liamkaplanmusic.com](https://liamkaplanmusic.com) was a great experience in improving my skills and knowledge of modern web technologies. I highly recommend using Gatsby.js and deploying to Netifly in order create beautiful and fast websites that are **free** to maintain. 

Getting my hands dirty building a website with modern web technologies left me extremely impressed by the tools developed in the last decade. Graphic design wise, I could not speak more highly of the Adobe suite and color pallate tools like [coolors.co](https://coolors.co/). Tools like React bootstrap help to make styling look consistently clean across your site with ease.

I only just entered the space, but it seems to me that it's never been easier to build beautiful and fast digital technology.