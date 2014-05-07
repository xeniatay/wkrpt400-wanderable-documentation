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

- frontend philosophy
- code guide


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

> E.g. Take a look at the networks panel in the Chrome Devtools on initial load

> Now take a look at it on page reload
(explain this)

#### Using the Asset Pipeline with Wanderable Site Components

Wanderable uses a customized strategy for asset precompilation, targeted towards the separate componenents of our site. Each of these components are targeted towards a different audience:

    - Public Site: potential users and guests looking for a registry
    - Internal Site: existing users who are using our actual product
    - Registry Layouts: existing users viewing others' registries, or guests who are purchasing gifts
    - Channels: white-label hotel registries and the customers they drive to us
    - Merchant Portal: merchants going through our self-serve Merchant Network
    - Admin: for internal use

Each component of the site generally has its own CSS and JS manifest and layout template in `apps/views/layouts`. 

TODO talk more about site componenets

For now, we will focus on how we handle asset precompilation for CSS and JS, with Bootstrap, LESS, Coffee and Rails. 

##### Manifest Files 

As mentioned earlier, Wanderable has many separate components that comprise the entire app. This requires us to handle asset loading and caching in a smarter way - for example, it does not make sense to load all of the styles for the public AND internal site if the user is merely a guest landing on a registry page to purchase a gift.

Thus, we allocated separate CSS and JS manifests for each of these components.

> Sprockets uses manifest files to determine which assets to include and serve. These manifest files contain directives - instructions that tell Sprockets which files to require in order to build a single CSS or JavaScript file. With these directives, Sprockets loads the files specified, processes them if necessary, concatenates them into one single file and then compresses them (if Rails.application.config.assets.compress is true). By serving one file rather than many, the load time of pages can be greatly reduced because the browser makes fewer requests. Compression also reduces file size, enabling the browser to download them faster.

Our manifest files are located at `app/assets/stylesheets/*-manifest.css` and `app/assets/javascripts/*-manifest.js`

Setting it up: in `application.rb`, we include the following line of code: 

    config.assets.precompile += %w( *-manifest.css *-manifest.js )

This automatically adds any filepath ending in `-manifest.css` or `-manifest.js` to the precompile path. The original manifests were `application.css` and `application.js`. 

Each manifest file contains [Sprocket directives](http://guides.rubyonrails.org/asset_pipeline.html#manifest-files-and-directives), which tell Sprockets which files to concatenate and minify into that specific manifest. 

We then include each compiled manifest into the layout they belong in. 

    <%= stylesheet_link_tag 'public-manifest' %>
    <%= javascript_include_tag "public-manifest" %>

A good rule of thumb when in doubt is just to grep for any file including `-manifest`.

##### CSS Manifests

You may notice that all our CSS manifest files only include a single `*-bundle` file, which seems redundant. 

This setup is to allow us to work with LESS. Wanderable relies on Bootstrap heavily, and Bootstrap makes heavy use of mixins, variables and other wonderful LESS features. 

Because manifest files are `.css` files, they are not able to compile from LESS the way we want to. Compiling across files is very important to us in the LESS workflow, to allow us to use the `@import` feature and other fancy things. Thus, our workaround is as such:

- A `*-manifest.css` file that includes only `-bundle` files. This gets precompiled and served to the user
- A `*-bundle.less` file that imports all the necessary LESS files across our assets for the particular site component
    - All newly created LESS files have to be included in their respective bundle through the `@import` function

The `global-bundle.less` is a special bundle that does not belong to any manifest. This is because it stands alone as the set of basic Wanderable styles 
    - bootstrap overrides
    - header styles
    - footer styles
    - type

Separating out the global bundle allows us to keep the bundle files [DRY](http://en.wikipedia.org/wiki/Don't_repeat_yourself) - the bundle can simply be included in other bundles, instead of repeating the `@import` statements over and over again.

We see this behaviour again in the `channel-bundle`, which imports the `internal-bundle`. 

*Note: the bundle/manifest setup works well in separating components, but may still cause repeated code in final compilation. It is up to the developer to decide on a tradeoff between DRY or clean code.*

###### Processing LESS, good practices and current conventions

Currently, *most* LESS files are named with the following format: 

_[filename]-[hyphenate]-[all]-[other]-[words].less 

The underscore was more important pre-less-rails, when we were compiling LESS files manually in our local environments. Underscores signify LESS partials, which we do not want compiled.

However, less-rails and the asset pipeline now handle that without any worry on our end. Our files are currently in a mixed syntax, some with and without prepended `_`, and some hyphenated and some underscored. It would be nice to someday agree on a convention and rename all files to follow it. 

##### Working with Bootstrap

Wanderable uses a heavily customized version of Bootstrap. You can see the exact variable overrides in `app/assets/stylesheets/less/global/bootstrap_overrides/_variables.less`

Note: our customization was done manually, not through the [customizer](http://getbootstrap.com/customize/), and thus is a little messy. It consists of variables which *may not* have originally been defined in Bootstrap's `variable.less`. *Nice to have: cleaning up the overrides file*

The Bootstrap CSS components most commonly used on the site are
    - Grid system and responsive utilities 
    - Typography
    - Tables
    - Forms
    - Buttons
    - Helper Classes

##### JS Manifests 

The JS manifest files are a lot more straightforward - they use the Sprocket directives to include either entire directories or specific files from certain directories, which are all commented in each manifest.

*Note the difference between require and require_tree* 
*jQuery and jQueryujs are required through gems*

One point to note is that the JS manifests do have dependencies - certain libraries like jQuery, Bootstrap, etc. have to be included at the top of the file before other plugins that use it. 

`ie-manifest` is a special manifest created to handle LTIE9 browsers. In those browsers (which Wanderable no longer supports), it triggers an unobstrusive header prompting the user to upgrade their browser.

We have started migrating to use coffee, but 90% of our files are still old JS.

Note: JS manifests are always included at the bottom of the body in a layout, to allow the user to interact with the site without needing to wait for all scripts to be done loading. 

Therefore, any inline page JS should be included like this:

    <% content_for :page_end do %>
        // code here
    <% end %>

##### Responsive

- cloudinary
- caching responsive
- nav
- nav breakpoints
- grid

### Workflow/helpful tools
- adding a new page
    - things to note 
        - js manifest
        - html
        - scope
        - bundle
color scheme

Note: Reading up on the Chrome Developer Tools is a *very good way* to learn what the browser is capable of, and how performance optimization is done. 
    - console debugger and stuff
    - inspect element
    - networks
    - timeline (jank, repainting)
    - resources (cookies, localhost, localstorage)

### Wanderable Site Layouts 
- public
- internal
- layout
- merchant
- admin

### Improvements
- webfontloader
- cutting out loads on layouts
- data-src for responsive images
