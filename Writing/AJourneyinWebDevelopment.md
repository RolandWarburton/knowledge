# My web development journey
Website development has been a constant force in my motivation to pursue computer science.
From the first line of code i ever wrote (in HTML) in 2017 to today i have always wanted to build websites (in a hobbyist sense).

It is now March 2020, 3 years since i seriously started learning about computers. and 1 year since i started university. In this time my personal folio site has undergone many versions and changes as my technical skills improve and my philosophy to development and software has changed over time. I hope to outline the major changes that the site has gone through as its purpose for existing and aesthetic has changed quite a bit in just the past 6 months.

# 2017. My first website
In 2017 i was in year 12 of Highschool and had some exposure to programming in a classroom. I absolutely hated it. By early-mid 2017 i started my Certificate III  in IT and digital media. My final assignment was to create a fake realty site. This was the first time i was expected to write any programming and deliver a finished product for assessment, it also happened to be the pilot year that this unit was running and i was mostly left to my own devices to educate myself on how to make a simple website.

> When writing my first website i fell into all the common traps that i see beginners falling into now.

At this stage i knew almost 0 CSS. And had barely understood HTML. Javascript was still a foreign or even possibly an unknown concept. One key memory from this experience was centering elements by using HTML tables. Something that i see people brand new to programming and creating websites doing today.

# 2018. Iteration 1 of my folio site
By 2018 i decided i wanted to have a site to showcase my skills as a programmer sometime through my diploma in IT. I found a HTML/CSS template and went to work on glueing bits of javascript together from stack overflow without knowing what it was doing.

By the end of 2018 i had a 'working website'. But as i was starting to develop programming concepts that i could apply to my folio website i soon scrapped it and started working on smaller web projects to build up my skills. such as making elements move like sliding navbars.

# 2019. Picking up key software skills
2019 was the year that i started my BA at swinburne. I had graduated from my diploma knowing C and some minimal C++ as well as learning javascript and php. The web based content that i learnt at university was all based in ES5 and was dated so it didn't include many of the modern web standard that have developed in the last 10 years, such as NODEJS, NPM, or ES6.

Mid to late 2019 marked halfway through my first year of university. At this time i was almost confident in self teaching myself programming and new technologies. During this period i was too focused on learning C# and OOP to be developing my folio site any further

Late 2019 i started really picking up web after my units subsided. Through the summer semester i put more emphasis on learning technical skills then completing a business unit i decided to enrol in. I finally started learning ReactJS. React was the first modern technology that i picked up.

# 2020. The modern web stack
After getting a react boilerplate running and successfully getting a version of my website that used react running i developed more of an interest in the concept of 'small sharp tools'. During my learning experience with Linux at the same time i decided that i should focus on learning the underlying platform beneath ReactJS. So i ran ```npm eject``` and started diving into webpack and its loaders, babel, dependencies, and npm packages. I had 'discovered' these technologies a 3-4 months prior yet didn't know their exact purpose or reason for existing.

# Making my site feature rich in ReactJS
Around January 2020 i converted my entire website to ReactJS. This was a lot of effort at the time as i went through many different iterations as i quickly caught up with what was considered 'best practice' in the react community.

## Component life cycle Vs React Hooks
The first major change i made to my site was my decision to use the new 'React Hook' rather than the old class based component system to manage the state of my components.

## Styling in ReactJS
This stage in my react development took quite a white of reading to understand. Somehow it was 'ok' to use inline styles. It was also perfectly fine to use traditional css style sheets. There were also new react/frontend framework specific ways of styling a website.

## CSS in JS
This was short lived for me. I decided this was not the best option for my website due to that i had spent a lot of time getting used to sass and making use of its mixins, nesting, and variables which WERE supported in CSS in JS but only through additional layers and frameworks (see styled components).

## Styled Components :nail_care:
Styled components were very interesting to me. I ended up re-writing my entire site in styled components just to get a feel for them. I remember thinking about all the ways of creating dynamic styles on the fly to address different problems that my site would have.

## Bundlephobia and overhead
This was a serious concern that i had during my entire phase of working on my personal site in React. I had some great help from a friend of my who referred me to someone who works in react for a full time job. After a conversation with him and ultimately coming away disappointed at his indifference to stacking up packages to solve problems that didn't seem warranted to adding dependencies i eventually moved away from react to research and possibly try other frameworks and explore concepts like Server Side Rendering SSR (and later SSG)


## React Vs NextJS Vs NuxtJS Vs Vue Vs Angular
Upon doing a lot of reading and research about web i decided that react was not the perfect solution for what i wanted my website to achieve. React **WOULD** have been able to do the things i wanted it to. However it had a lot of overhead.

## React SSR
My first attempt to make the website more streamlined and lightweight was to try and add Server Side Rendering to the site, where the server would compile initial HTML and send it to the client, and then react would come in once its bundle had loaded and attach itself to the DOM. At which point it could hydrate the DOM with dynamic content. The only problem was that i didn't have any dynamic content to render in the first place! This tipped me off to go look for other solutions.

# Small Sharp Tools
Finally the last part of the websites development!

Upon reading a lot about SSR and comparing React and its main SSR counterpart NextJS. I decided to put NextJS on the list of things i will learn one day. And they threw out the front end framework entirely. I then pivoted my philosophy from wanting to create a visually impressive and feature rich website. To something that was small, lightweight, and easy to run, maintain, and deliver. Than meant going to some extremes. But considering is my own personal site i could take it in any interesting direction i wanted.

Taking what i had learnt from bundling JS in webpack i thought that i could one day remove the JS bundle entirely and ship a JS free website. This turned out to not be possible without some very hacky solutions.

I replaced webpack by 'loading' images and other media with fs-extra - an extension of the node fs library. 

Sass is simply loaded with node-sass - A high speed scss/sass compiler and dumped directly into a distribution/public folder. node-sass comes with its own built in compressor which reduces the dependencies from 4 (style-loader, post-css, sass-loader, css-loader) to just one, a perfect example of simplifying the website and making its development more high performance while maintaining all bare-bones functionality that i was aiming for.

HtmlWebpackPlugin is replaced by relying on EJS, my templating language of choice to render out the page using its own engine. And then using a generic html minifying package to remove comments and whitespace to make it as small as possible for shipping.

To fill in missing features such as dynamic routing in React i wrote several build functions that map through the websites javascript files that exported template literals with appropriate content already baked in. I can build navigation links, file structure, retrieve content, and add any other static functionality that an easy to read/navigate website should include using just NODE.

To maintain the sites content i plan to use node-fetch as a drop in replacement for response, which has now recently been deprecated, and then pull content that i write from gists. All content that i write, inside and outside of the website is in markdown and providing them to the website with any of those 3 techniques will result in the same HTML output so i am not bothered at the end of the day. I hope to one day find a way to pull content from my private repo so that i can take notes and programming snippits from my university work which has to maintain private in accordance with my universities policies.

Lastly the biggest improvement in performance that i saw from moving to this unique development process was in changing the development server from webpack-dev-server to using ExpressJS as a static file server with nodemon and concurrently to run both nodemon and Express at the same time to both host a local server and monitor the files at the same time. This maintains all the original functionality webpack-dev-server provided to me except with the reduces overhead. Bringing the build time to under 1s compared to ~4s using webpack.
