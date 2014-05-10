# Wanderable: Frontend Notes

> Notes
1500 and 2000 words
Title Page (Section 3.2)
Letter or Memorandum of Submittal (Section 3.3)
Analysis (Section 3.12 if you are submitting a user manual or a non-analytical report)
Recommendations, if appropriate (Section 3.9)
Acknowledgments, if appropriate (Section 3.11)
Completed employer evaluation form (.pdf) (Section 2.2)

## 500 word Analysis 

    Include a Title Page and Letter or Memorandum of Submittal with your Analysis. 
    For example, if you write a user’s manual, you might include the following: 
    the purpose of the manual
    the characteristics of a good user manual
    possible improvements to the manual
    reasons why the software described in the manual is needed and the problems it solves
    specific examples illustrating the impact of the software or of the manual on the user

## Table of Contents

## Development Stack

The Wanderable site is built using:

- [Rails (3.1.3)](http://guides.rubyonrails.org/)
> Ruby on Rails is a full-stack web framework optimized for programmer happiness and sustainable productivity. It encourages beautiful code by favoring convention over configuration.

- [LESS (1.7.0)](http://lesscss.org/) via the [less-rails gem](https://github.com/metaskills/less-rails) 
> A *gem* is a Ruby software package that provides external functionalities to an existing Rails project. 
> Less is a CSS-preprocessor, meaning that it extends the CSS language, adding features that allow variables, mixins, functions and many other techniques that allow you to make CSS that is more maintainable, themable and extendable. 

- [CoffeeScript](http://coffeescript.org/) via the [coffee-rails gem](https://github.com/rails/coffee-rails)
> CoffeeScript is a little language that compiles into JavaScript. CoffeeScript is an attempt to expose the good parts of JavaScript in a simple way. 

- [Bootstrap (3.1.1)](http://getbootstrap.com/)
> The most popular front-end framework for developing responsive, mobile first projects on the web.

- [jQuery](http://jquery.com/) via the [jQuery-rails gem](https://github.com/rails/jquery-rails)
> jQuery is a fast, small, and feature-rich JavaScript library.

Although the app does not rely solely on these technologies, a good grasp of them is enough for basic development on Wanderable's frontend.

### Wanderable's Asset Pipeline

> The asset pipeline provides a framework to concatenate and minify or compress JavaScript and CSS assets. It also adds the ability to write these assets in other languages and pre-processors such as CoffeeScript, Sass and ERB.

The [Rails Asset Pipeline](http://guides.rubyonrails.org/asset_pipeline.html) is a crucial step in our development workflow. Below is a brief summary of pipeline functionality followed by a guide to Wanderable-specific pipeline behaviour. **Note:** The official Rails guide provides a much richer explanation and should be read for full understanding of the pipeline. 

#### Asset Precompilation: Concatenation and Minification 

> The first feature of the pipeline is to concatenate assets, which can reduce the number of requests that a browser makes to render a web page. Web browsers are limited in the number of requests that they can make in parallel, so fewer requests can mean faster loading for your application.

> The second feature of the asset pipeline is asset minification or compression. [...]

Through the Asset Pipeline, Sprockets (a Ruby library) concatenates and minifies all specified CoffeeScript/JS files into one master `.js` file, and all Less files into one master `.css` file. This process is called asset precompilation.

Wanderable's asset precompilation is done automatically through Heroku when deploying a new version of the app to a Heroku server.

##### How the pipeline behaves in development

On a local development server, assets are not concatenated or minified, so that debugging by line number is easy. However, this means that the local server makes *many* requests upon page load for every single file included in the precompile path.

**Note:** The above behaviour can be slow and frustrating. To use precompiled files in development, change the following line in `application.rb` to *false*:

    # Expand the lines which load the assets
    config.assets.debug = true 

Keep in mind that this will affect frontend development workflow and you should not commit this change.

##### How the pipeline behaves in production 

Assets are always precompiled on Wanderable's production server. 

On a user's initial visit, both the master CSS and JS files are requested from the server and subsequently cached by the browser. Here is an example of the browser requests on Wanderable's homepage:

> TODO insert image

Caching asset files is beneficial to a user because it decreases site load time on all of their future visits. Here is the same page as shown above, upon page refresh. 

> TODO insert image

Observe that the cached assets resulted in a 0.6s decrease in load time on Wanderable's homepage. 

### Asset Precompilation with CoffeeScript and Less

> The third feature of the asset pipeline is it allows coding assets via a higher-level language, with precompilation down to the actual assets.

Wanderable uses the pipeline to allow live compilation of CoffeeScript and Less in development mode. This integration happens automatically through the `less-rails` and `coffee-rails` gems.

**Advantages of compiling CoffeeScript and Less through the pipeline:**

- Do not need to keep compiled `.less -> .css` and `.coffee -> .js` files in the repository
- Prevents merge conflicts caused by committing compiled CSS and JS 

**Note**: Compiling through pipeline might cause your local server to fail silently when Rails encounters a Less or CoffeeScript error. 

### Custom Asset Pipeline Behaviour 

**How does Wanderable handle asset precompilation?**

Instead of having a master CSS and JS file (called *manifest* files) across the entire site, Wanderable has multiple manifest files based on site component. 

**What is a site component?**

A site component is a major part of the site that has:

- A layout template in `views/layouts/`
- A CSS manifest file in `app/assets/stylesheets/`
- A JS manifest file in `app/assets/javascripts/` 

Wanderable's app is split into site components because each component is used by a different target audience. 

**What are the Wanderable site components?**

- Public Site 
    - For potential users and guests looking for a registry
    - `pubsite_layout.html.erb`, `public-manifest.css`, `public-manifest.js`
- Internal Site 
    - For couples who log in and edit their registries
    - `application.html.erb`, `internal-manifest.css`, `internal-manifest.js`
- Registry Layouts 
    - For guests making a gift purchase, or existing users browsing other registries
    - `registry_layout.html.erb`, `registry-layouts-manifest.css`, `layout-manifest.js`
- Channels 
    - For guests referred to us through our white-label hotel partnerships 
    - `channel.html.erb`, `channel-manifest.css`, `channel-manifest.js`
- Merchant Portal 
    - For merchants going through our self-serve Merchant Network
    - `merchant_portal.html.erb`, `merchant-manifest.css`, `merchant-manifest.js`
- Admin 
    - For internal use
    - `admin_layout.html.erb`, `admin-manifest.css`, `admin-manifest.js`

**Why have separate manifest files for each site component?**

The purpose of a manifest file is to speed up the load time of our site. We know that none of our users typically access every part of our site. 

> E.g. A guest visits a registry to make a gift purchase. They do not care to load the assets containing styles and scripts for the merchant portal or the internal site. 

Thus, splitting up these manifests based on site component and target audience allows us to reduce load time for everyone who visits Wanderable. 

### How Manifest Files Work

Manifest files contain [Sprocket directives](http://guides.rubyonrails.org/asset_pipeline.html#manifest-files-and-directives), which tell Sprocket which files to concatenate and minify into a compiled version of the manifest. 

In a default Rails application, manifest files are named `application.css` and `application.js`. 

To override that behaviour and allow automatic precompilation of multiple manifest files, we added the following line in `application.rb`:

    config.assets.precompile += %w( *-manifest.css *-manifest.js )

This line automatically adds any files ending in `-manifest.css` or `-manifest.js` to the precompile path.

Custom manifests are then included in the layout they belong in:

    <%= stylesheet_link_tag 'public-manifest' %>
    <%= javascript_include_tag 'public-manifest' %>

#### CSS Manifests

Notice that all our CSS manifest files only include a single `*-bundle` file, which seems redundant. 

This is a workaround to accommodate Less pre-processing features while still using the asset pipeline for precompilation. 

**An explanation of the *-bundle.less files*

When a Less file is included in a manifest by a Sprocket directive, it is compiled individually. This means that it does not have access to mixins and variables defined in other Less files. 

This is very restrictive and undesirable behaviour. To work around this, we use `@import` to consolidate all required files into a *bundle*. This bundle is then included in its corresponding manifest and compiled using Sprocket. 

**Important:** In order to include a new Less file on the site, it has to be added to the appropriate `bundle` using the `@import` function. 

**About global-bundle.less**

`global-bundle.less` is a special bundle that does not belong to any one manifest. It stands alone as a set of basic Wanderable styles and contains the following:

- Bootstrap overrides
- Header styles
- Footer styles
- Type styles

This bundle was separated from the rest because it is re-used by so many other bundles. The global bundle is simply `@import`ed into other bundles, so that all bundles are kept [DRY](http://en.wikipedia.org/wiki/Don't_repeat_yourself). 

We see this behaviour again in the `channel-bundle`, which imports `internal-bundle`. 

#### Working with Bootstrap

Wanderable uses a heavily customized version of Bootstrap. You can see the exact variable overrides in `app/assets/stylesheets/less/global/bootstrap_overrides/_variables.less`

Note: our customization was done manually, not through the [customizer](http://getbootstrap.com/customize/), and thus is a little messy. It consists of variables which *may not* have originally been defined in Bootstrap's `variable.less`. *Nice to have: cleaning up the overrides file*

The Bootstrap CSS components most commonly used on the site are
    - Grid system and responsive utilities 
    - Typography
    - Tables
    - Forms
    - Buttons
    - Helper Classes

#### JS Manifests 

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

However, inline JS is strongly discouraged - the idea is that each page that requires JS should have its own JS file, scoped using its `body-class`. JS file naming conventions: currently we use _ as delimiters, and good practice is to name the JS file the same thing as the view file. This gets confusing if there are two views in different directories with the same name (_form, etc.) so use your discretion and grep accordingly.

We also use [readyselector](http://railsapps.github.io/rails-javascript-include-external.html) to scope our JS. This is done using the 

    <% content_for :body_class do %> page-name page-name-js <% end %>

`.page-name` will usually be used to scope CSS, and `page-name-js` will be used to scope JS like this:
    
    $('.page-name-js').ready(function() {
        // code goes here
    });


We scope all of our CSS and JS to the best of our abilities because one of the drawbacks to concatenation is that every page shares the same JS. Thus, when not properly scoped, broken JS in one file can affect the whole site and break all of the JS. 

This can be seen most commonly when DOM manipulation is happening, e.g.

    var box = $('#box-container'),
        squares = box.find('.squares');

    squares.last().remove();

    // if #box-container does not exist, squares will be undefined and this code will break

#### Responsive

Media queries are used to build responsive sites. 
E.g.
    
    .container {
        width: 1180px;
    }

    @media (min-width: 600px) and (max-width: 800px) {
        .container {
            width: 800px;
        }
    }

In LESS, both the syntax above and the following syntax is supported: 

    .container {
        width: 1180px;

        @media (min-width: 600px) and (max-width: 800px) {
            width: 1180px;
        }
    }

At Wanderable, we use the second LESS-only syntax, because it is cleaner to wrap all of the styles related to a selector in its own braces. This also means that the hierachy of styles will be as follows

1. Default styles for a selector
1. Element state styles
1. Pseudo elements, if any
1. Media queries, if any
    - Also include nested styles for element states or pseudo elements at that viewport
1. Nested elements, if any

    .container {
        width: 1180px;

        &:hover {
            color: red;
        }

        &:before,
        &:after {
            content: '/';
            display: block;
        }

        @media @phone {
            width: 800px;

            &:hover {
                color: black;
            }

            &:before,
            &:after {
                display: none;
            }
        }

        .child-container {
            width: 50%;
        }
    }

Wanderable's site is fully responsive, largely relying on [Bootstrap's responsive grid](http://getbootstrap.com/css/#grid) and responsive utility classes. 

A simple philosophy behind building responsive pages is to keep in mind how the page should look on mobile even when building out the desktop markup. Also, avoid including classes if they are uncessary. Typically, this means writing classes like this:

    <div class="col-xs-12 col-sm-6 col-md-4 col-lg-3"></div>

or this:

    <div class="col-xs-12 col-sm-3"></div>

The code above depicts a column of content - when there is more screen estate, the column spans less of the whole page, and increases accordingly according to the viewport. In the second example, we want the column to be 1/3 of the container, for any screen size > `sm`. Thus, we do not include the `md` and `lg` classes.

Bootstrap's responsive philosophy is mobile-first. Unfortunately, we don't really use this at Wanderable - responsive is a feature, but not our priority.  

Currently, our LESS is written desktop-first, and by desktop I mean {768px - 991px}. This is mostly because IE8 does not support media queries, and writing desktop-first LESS will allow default fallbacks to how the site should look in desktop view.

However, after most of the responsive LESS for the public site was written, we decided to drop support for IE8. This grants us the freedom to write media queries in any order we want! 
 

We have a bunch of helper variables built in on top of Bootstrap's media query variables. This was done because their media queries felt clunky and repetitive - and were unintuitive. 

    // TODO: clean this crazy stuff up
    @screen-phone-max: @screen-xs-max;
    @screen-tablet-max: @screen-sm-max;
    @screen-desktop-max: @screen-md-max;

    @phone-v: ~"(max-width: 319px)";
    @phone-h: ~"(min-width: 320px) and (max-width: 479px)"; // max-width: 479px
    @phone: ~"(max-width: 479px)"; // max-width: 479px
    @tablet: ~"(min-width: @{screen-xs}) and (max-width: @{screen-xs-max})"; // min-width: 480px, max-width: 767px
    @desktop: ~"(min-width: @{screen-sm}) and (max-width: @{screen-sm-max})"; // min-width: 768px, max-width: 991px
    @desktop-lg: ~"(min-width: @{screen-desktop})"; // min-width: 992px

    @lt-tablet: @phone; // max-width: 479px;
    @lt-desktop: ~"(max-width: @{screen-xs-max})"; // max-width: 767px
    @lt-desktop-lg: ~"(max-width: @{screen-md-max})"; // max-width: 991px

    @desktop-only: ~"(min-width: @{screen-sm})"; // min-width: 768px;

I will be the first to admit that our queries are also cluttered, but they are more straightforward. As you can see, there is a TODO to clean it up.

I also added some responsive utilities in `global/_responsive-utilities.less`. These were basically to handle the `@phone-v` cases which bootstrap does not support.

Basically, the queries I added just simplifies Bootstrap's syntax and adds a layer of meaning (phone, tablet, etc.) to the queries. 

For example, Wanderable's media query:
    
    @media @tablet {
        // responsive code here
    }

Bootstrap's version:

    @media (min-width: @screen-xs) and (max-width: @screen-sm-max) {
        // responsive code here
    }

One caveat is this: media queries operate by inheritance, the same as any other CSS rule.

So this means that 
    
    @media @lt-desktop { // phone and tablet
        color: red;
    }

    @media @phone {
        color: blue;
    }

The color will be blue when the viewport is phone small. This is despite @lt-desktop encompassing both phone and tablet.

    @media @phone {
        color: blue;
    }

    @media @lt-desktop { // phone and tablet
        color: red;
    }

Colour will always be red, because red overwrites blue.

The right way: 

    @media @phone {
        color: blue;
    }

    @media @tablet {
        color: red;
    }

Different colors :) 

One more thing: @grid-breakpoint is a bootstrap variable used for the responsive nav, it is manually defined and should be changed accordingly if needed.

##### Doing responsive with Cloudinary 

Look at `global/_responsive-img-mixins.less`.

Basically, wrote a bunch of mixins to tie in with Cloudinary's API so that LESS compiles responsive images for us automatically.

Also made use of Cloudinary's image processing for the public site.

Else, responsive images are usually done like this 

    .hero-unit {
        background-image: url('image-url.png');
        background-size: cover;
        background-repeat: no-repeat;

        @media @phone { background-image: url('image-url-phone.png'); }
        @media @tablet { background-image: url('image-url-tablet.png'); }
    }

Yes, it's very tedious.

The reason this works is because background images are then loaded conditionally depending on the viewport. 

Background images which aren't supposed to be loaded will then only be loaded when necessary - usually only when the user resizes the window on purpose. 

Cool fact: if an element with a background image is loaded, most modern browsers are also smart enough to not load the background image at all.

Also, this can only be done with background images, not `<img>` tags.

### Additional notes

Here's Wanderable's color palette as defined in `_variables.less`

#### Smooth-scrolling plugin

`global/smooth-scroll.js`
Enables smooth-scrolling to any anchor link on the same page. 

Simply add attribute "data-smoothscroll='true'" to any anchor link to enable smooth scrolling

    <a href="#signup-form" data-smoothscroll="true">Go to signup</a>

#### Parallax image plugin

`global/parallax-scrolling.js`

Simply add class `.parallax-scroll` to any container element with a background image.

This class is used by both CSS and JS.

#### Adding a new page

The workflow for adding a new page is as follows:

In the `.html.erb` file, if it requires any custom CSS or JS, scope it using
    
    <% content_for :body_class %> page-sample page-sample-js <% end %>

Create a CSS file in the corresponding directory (for its component). 

Include the CSS file in the appropriate bundle. 

Create a JS file in the corresponding directory (for its component). It will magically be included on the page and you can check this by viewing all the scripts that are being included on the page in development mode. Remember to scope the JS.

### Workflow/helpful tools

A customized `.gitconfig` file is always a nice-to-have, it allows for color highlighting in diffs, prettier git logs. 

Changing your shell (bash, zsh, fish) also allows you to add cool stuff in your prompt like git branch statuses and the current hash. 

Sublime Text 3 is a great open-source text editor, best feature is fuzzy finding, which allows you to `cmd + p` and search for any filename without needed to autocomplete. It has Vim support in the form of Vintage and Vintageous packages, but note that these packages are still being developed and it does not act exactly like Vim although it tries its best.

CodeKit is a great tool for compiling LESS, Coffee, or any other sort of pre-processed languages. It costs $30 and hasn't been necessary for Wanderable since we started compiling LESS and Coffee through Rails, but it is good to know. It also offers Linting on the fly!

using the Networks tab in the [Chrome Developer Tools](https://developers.google.com/chrome-developer-tools/docs/network)

Note: Reading up on the Chrome Developer Tools is a *very good way* to learn what the browser is capable of, and how performance optimization is done. 
    - console debugger and stuff
    - inspect element
    - networks
    - timeline (jank, repainting)
    - resources (cookies, localhost, localstorage)

- frontend philosophy
- code guide

##### Processing LESS, good practices and current conventions

Currently, *most* LESS files are named with the following format: 

_[filename]-[hyphenate]-[all]-[other]-[words].less 

The underscore was more important pre-less-rails, when we were compiling LESS files manually in our local environments. Underscores signify LESS partials, which we do not want compiled.

However, less-rails and the asset pipeline now handle that without any worry on our end. Our files are currently in a mixed syntax, some with and without prepended `_`, and some hyphenated and some underscored. It would be nice to someday agree on a convention and rename all files to follow it. 

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
