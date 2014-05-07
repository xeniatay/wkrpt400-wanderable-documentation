## Wanderable Front End 
========

### Table of Contents
1500 and 2000 words
Title Page (Section 3.2)
Letter or Memorandum of Submittal (Section 3.3)
Analysis (Section 3.7; Section 3.12 if you are submitting a user manual or a non-analytical report)

Recommendations, if appropriate (Section 3.9)

Acknowledgments, if appropriate (Section 3.11)

Completed employer evaluation form (.pdf) (Section 2.2)

### 500 word Analysis 

Include a Title Page and Letter or Memorandum of Submittal with your Analysis.

For example, if you write a user’s manual, you might include the following:

the purpose of the manual
the characteristics of a good user manual
possible improvements to the manual
reasons why the software described in the manual is needed and the problems it solves
specific examples illustrating the impact of the software or of the manual on the user

### How Things Work

#### Frontend Tech Stack

At Wanderable, we use the following technologies:
- Ruby (1.9.3)
- [Rails (3.1.3)](http://guides.rubyonrails.org/)
- [LESS (1.7.0)](http://lesscss.org/) via the [less-rails gem](https://github.com/metaskills/less-rails)
- Coffee via the [coffee-rails gem](https://github.com/rails/coffee-rails)
- [Bootstrap (3.1.1)](http://getbootstrap.com/)
- jQuery via the [jQuery-rails gem](https://github.com/rails/jquery-rails).

##### Rails Asset Pipeline

Wanderable upgraded to Rails 3.1.3 in March 2014, enabling usage of the [Rails Asset Pipeline](http://guides.rubyonrails.org/asset_pipeline.html) in our development workflow. 

The Asset Pipeline allows us to concatenate all specified LESS or Coffee files into master CSS and JS files, which load in a single request upon initial site visit. These files are then cached in the user's browser, and cost significantly less load time throughout the rest of a user's browsing experience or on return visits. 

> [Sprockets](https://signalvnoise.com/posts/1587-introducing-sprockets-javascript-dependency-management-and-concatenation) was originally introduced by Basecamp in 2009. Sprockets is a Ruby library that preprocesses and concatenates JavaScript source files.

The process of concatenation and minification is called asset precompilation. This is done through Heroku when we deploy a new version of our app.

    > The asset pipeline provides a framework to concatenate and minify or compress JavaScript and CSS assets. It also adds the ability to write these assets in other languages and pre-processors such as CoffeeScript, Sass and ERB.

    > The first feature of the pipeline is to concatenate assets, which can reduce the number of requests that a browser makes to render a web page. Web browsers are limited in the number of requests that they can make in parallel, so fewer requests can mean faster loading for your application.

    > Sprockets concatenates all JavaScript files into one master .js file and all CSS files into one master .css file. [...]

    > The second feature of the asset pipeline is asset minification or compression. For CSS files, this is done by removing whitespace and comments. For JavaScript, more complex processes can be applied. You can choose from a set of built in options or specify your own.

    > The third feature of the asset pipeline is it allows coding assets via a higher-level language, with precompilation down to the actual assets. Supported languages include Sass for CSS, CoffeeScript for JavaScript, and ERB for both by default.

Here is an example using the Networks tab in the [Chrome Developer Tools](https://developers.google.com/chrome-developer-tools/docs/network)

    E.g. Take a look at the networks panel in the Chrome Devtools on initial load

    Now take a look at it on page reload
    (explain this)

#### How we use the Asset Pipeline

Wanderable uses a customized strategy for asset precompilation, targeted towards the separate componenents of our site. Each of these components are targeted towards a different audience:

    - Public Site: potential users and guests looking for a registry
    - Internal Site: existing users who are using our actual product
    - Registry Layouts: existing users viewing others' registries, or guests who are purchasing gifts
    - Channels: white-label hotel registries and the customers they drive to us
    - Merchant Portal: merchants going through our self-serve Merchant Network
    - Admin: for internal use

You will learn more about the separate components in section TODO. 

For now, we will focus on how we handle asset precompilation for CSS and JS, with Bootstrap, LESS, Coffee and Rails. 

##### LESS, Bootstrap and the Asset Pipeline

As mentioned earlier, Wanderable has many separate components that comprise the entire app. This requires us to handle asset loading and caching in a smarter way - for example, it does not make sense to load all of the styles for the public AND internal site if the user is merely a guest landing on a registry page to purchase a gift.

Thus, we allocated separate CSS and JS manifests for each of these components.

> Sprockets uses manifest files to determine which assets to include and serve. These manifest files contain directives - instructions that tell Sprockets which files to require in order to build a single CSS or JavaScript file. With these directives, Sprockets loads the files specified, processes them if necessary, concatenates them into one single file and then compresses them (if Rails.application.config.assets.compress is true). By serving one file rather than many, the load time of pages can be greatly reduced because the browser makes fewer requests. Compression also reduces file size, enabling the browser to download them faster.





- Concatenation and minifying through Sprockets
- Less requests
- Cached 
- Using manifests and bundles
- cloudinary
- caching responsive
The biggest reason for using the Asset Pipeline is concatenation and minifying of all assets on the site. 
Fewer requests automatically mean 

    E.g.

- Brief summary DONE 
- assets pipeline DONE
- coffee
- compiling less
- links to relevant rails docs
- responsive
- frontend philosophy
- code guide

Work report: 
### Using Bootstrap at Wanderable
- overrides and caveats
     - nav breakpoint
     - helper responsive classes
- Conventions 

### Workflow/helpful tools
- adding a new page
    - things to note 
        - js manifest
        - html
        - scope
        - bundle

Note: Reading up on the Chrome Developer Tools is a *very good way* to learn what the browser is capable of, and how performance optimization is done. 
    - console debugger and stuff
    - inspect element
    - networks
    - timeline (jank, repainting)
    - resources (cookies, localhost, localstorage)

### Wanderable Site Components
- public
- internal
- layout
- merchant
- admin

### Improvements
- webfontloader
- cutting out loads on layouts
- data-src for responsive images
