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

## Wanderable's Asset Pipeline

> The asset pipeline provides a framework to concatenate and minify or compress JavaScript and CSS assets. It also adds the ability to write these assets in other languages and pre-processors such as CoffeeScript, Sass and ERB.

The [Rails Asset Pipeline](http://guides.rubyonrails.org/asset_pipeline.html) is a crucial step in our development workflow. Below is a brief summary of pipeline functionality followed by a guide to Wanderable-specific pipeline behaviour. **Note:** The official Rails guide provides a much richer explanation and should be read for full understanding of the pipeline. 

### Asset Precompilation: Concatenation and Minification 

> The first feature of the pipeline is to concatenate assets, which can reduce the number of requests that a browser makes to render a web page. Web browsers are limited in the number of requests that they can make in parallel, so fewer requests can mean faster loading for your application.

> The second feature of the asset pipeline is asset minification or compression. [...]

Through the Asset Pipeline, Sprockets (a Ruby library) concatenates and minifies all specified CoffeeScript/JS files into one master `.js` file, and all Less files into one master `.css` file. This process is called asset precompilation.

Wanderable's asset precompilation is done automatically through Heroku when deploying a new version of the app to a Heroku server.

### How the pipeline behaves in development

On a local development server, assets are not concatenated or minified, so that debugging by line number is easy. The tradeoff for this behaviour is that the local server makes *many* requests upon page load. These requests are for every single file included in the asset pipeline precompile path.

**Tip:** The above behaviour can be slow and frustrating. To turn off debug mode and request all assets in concatenated form, change the following line in `application.rb` to *false*:

    # Expand the lines which load the assets
    config.assets.debug = true 

Keep in mind that this will affect frontend development workflow. This is only a workaround for quicker local server response, and should not be committed to the repo.

### How the pipeline behaves in production 

Assets are always precompiled on Wanderable's production server. 

On a user's initial visit, both the master CSS and JS files are requested from the server. Here is an example of the browser requests on Wanderable's homepage:

> TODO insert image

Upon the initial visit, our asset files will be cached by the user's browser.When this same user visits another Wanderable page, their page load time will be significantly decreased. This is because the browser no longer has to load the cached asset files. 

Here is an example of the browser requests on the same page as above, upon page refresh:

> TODO insert image

Observe that the cached assets resulted in a 0.6s decrease in load time on Wanderable's homepage. This decrease in load time will be especially valuable on a slow or mobile connection.

## Asset Precompilation with CoffeeScript and Less

> The third feature of the asset pipeline is it allows coding assets via a higher-level language, with precompilation down to the actual assets.

Wanderable uses the pipeline to allow live compilation of CoffeeScript and Less in development mode. This integration happens automatically through the `less-rails` and `coffee-rails` gems.

**Advantages of compiling CoffeeScript and Less through the pipeline:**

- Do not need to keep compiled `.less -> .css` and `.coffee -> .js` files in the repository
- Prevents merge conflicts caused by committing compiled CSS and JS 

**Note:**: Compiling through the pipeline means that syntax errors from CoffeeScript and Less will appear in the stack trace from the server, and may throw 500 errors.

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

## How Manifest Files Work

Manifest files contain [Sprocket directives](http://guides.rubyonrails.org/asset_pipeline.html#manifest-files-and-directives), which tell Sprocket which files to concatenate and minify into a compiled version of the manifest. 

In a default Rails application, manifest files are named `application.css` and `application.js`. 

To override that behaviour and allow automatic precompilation of multiple manifest files, we added the following line in `application.rb`:

    config.assets.precompile += %w( *-manifest.css *-manifest.js )

This line automatically adds any files ending in `-manifest.css` or `-manifest.js` to the precompile path.

Custom manifests are then included in the layout they belong in:

    <%= stylesheet_link_tag 'public-manifest' %>
    <%= javascript_include_tag 'public-manifest' %>

### CSS Manifests

Notice that all our CSS manifest files only include a single `*-bundle` file, which seems redundant. 

This is a workaround to accommodate Less pre-processing features while still using the asset pipeline for precompilation. 

#### An explanation of the *-bundle.less files

When a Less file is included in a manifest by a Sprocket directive, it is compiled individually. This means that it does not have access to mixins and variables defined in other Less files. 

This behaviour is restrictive and undesirable. As a workaround, we used `@import` to consolidate all required files into a *bundle*. This bundle is then included in its corresponding manifest and compiled using Sprocket. 

**Important:** In order to include a new Less file on the site, it has to be added to the appropriate bundle using the `@import` function. 

**A note on global-bundle.less**

`global-bundle.less` is a special bundle that does not belong to any one manifest. It stands alone as a set of basic Wanderable styles and contains the following:

- Bootstrap overrides
- Header styles
- Footer styles
- Type styles

This bundle was separated from the rest because it is re-used by so many other bundles. The global bundle is simply `@import`ed into other bundles, so that all bundles are kept [DRY](http://en.wikipedia.org/wiki/Don't_repeat_yourself). 

We see this behaviour again in the `channel-bundle`, which imports `internal-bundle`. 

### JS Manifests 

Wanderable's JS manifests use the following Sprocket directives to include files: 
- `require_tree`: includes all the JS files in the specified directory, in no particular order
- `require`: includes the specific JS file

#### File Hierachy in JS Manifests

Most Javascript files have dependencies, which means that they rely on code from other libraries. Thus, important libraries like `jQuery` and `Bootstrap` have to be included at the top of the manifest. 

Here is a general hierachy to follow when setting up a new manifest:

1. jQuery and jQuery-rails 
2. Global libraries (bootstrap, jqueryUI, underscore) 
3. Validation plugins (need to be included in specific order)
4. Global directory 
5. Plugin directory corresponding to current manifest, e.g. `javascripts/public/plugins`
6. Main directory corresponding to current manifest, e.g. `javascripts/public`

Refer to an existing manifest to see how this is done with Sprocket directives.  
*Note*: `jquery` and `jquery_ujs (jQuery-rails)` are added to the app through Rails gems

**A note on ie-manifest.js**

`ie-manifest` is a special manifest created to handle LTIE9 browsers. In those browsers, an unobstrusive header appears, prompting the user to upgrade their browser.

### readyselector.js

[ReadySelector](https://github.com/Verba/jquery-readyselector) is a jQuery plugin that provides a nice syntax to write page-specific script. This idea was taken from [*Unholy Rails*](http://railsapps.github.io/rails-javascript-include-external.html).

Scoping JS is important when working with concatenated code. Unscoped code with a single syntax error has the potential to break the whole manifest file. 

**How do I scope my javascript?**

1. Add a unique class name to a container element that will be used to scope the script
2. Use ReadySelector to check for this element and wrap it in a `ready()` function

Example ReadySelector syntax: 

    $('.wrapper-form-container').ready(function() {
        // Event handlers and element-specific
        // code goes here
    });

ReadySelector will check whether `.wrapper-form-container` exists, and will only execute the scoped scripts if this element is done loading. 

### Creating a new Wanderable page

Putting it all together, these are the steps you take to create a new page on Wanderable:

**HTML/View**

- First, add a `body_class` to the page in the view. 
    
``` 
    <% content_for :body_class do %> page-sample page-sample-js <% end %>
    <% content_for :content do %> 
      // page content here
    <% end %>
```

**Less**

- Next, create a `.less` file and place it in the appropriate component directory 
    - File example: `app/assets/stylesheets/less/global/_search-page.less`

- In this Less file, nest all new styles in a `body_class` wrapper
    - An example of a Less file for a page with the body_class `.page-sample`

```
    // styles for page_sample.html.erb
    // ------------------------------

    .page-sample {
        // write styles here
    }
```

- Include this Less file in the appropriate component bundle
    - E.g. `@import "less/global/_header";`

**CoffeeScript/Javascript**

- Next, create a `.coffee` or `.js` file and place it in the appropriate component directory 

- In this Coffee or JS file, scope the page-specific code like this: 

```
    /*
      JS for page_sample.html.erb
      Scope: .page-sample-js
    */

    $('.page-sample-js').ready(function() {
        // write scripts here
    });
```

This JS file will automatically be included on the site by the component's manifest. 

Note that Wanderable has started using CoffeeScript for JS, but 90% of the existing script files are older and thus written in Javascript. 

**Writing inline Javascript in a view**

In a site component layout, the JS manifest is always included at the bottom of the body. This allows the user to interact with the site without waiting for scripts to load. 

Therefore, all inline page JS should be included like this:

    <% content_for :page_end do %>
        <script>
            // code here
        </script>
    <% end %>

Note that inline JS is strongly discouraged. Any page that requires JS should have its own page-scoped JS file. 

## Wanderable's Customized Bootstrap

Wanderable uses Bootstrap with customized variables that are listed in `app/assets/stylesheets/less/global/bootstrap_overrides/_variables.less`.

**Note:** This customization was done manually, instead of using the [customizer](http://getbootstrap.com/customize/). As a result, the file is a little disorganized and consists of variables which were not originally defined in Bootstrap's `variable.less`. *It would be nice to clean this up.*

## Responsive Development

**Using Media Queries**

Media queries are used to build responsive sites. An example of a media query using Less syntax:
    
    .container {
        width: 1180px;

        @media (min-width: 600px) and (max-width: 800px) {
            width: 800px;
        }
    }

Writing media queries like this can quickly lead to a mess of complicated, nested rules. To prevent this, organize your Less rules using the following convention: 

1. Element styling
1. Element state (active, hover) styling
1. Pseudo elements, if any
1. Media queries, if any
    - Include element state styling or pseudo element styling 
1. Nested elements, if any

Here is an example of how to organize your Less rules:

    .container {
        // element styling
        width: 1180px;

        // element state
        &:hover {
            color: red;
        }

        // pseudo elements
        &:before,
        &:after {
            content: '/';
            display: block;
        }

        // media query
        @media @phone {
            width: 800px;

            // element state
            &:hover {
                color: black;
            }

            // pseudo element
            &:before,
            &:after {
                display: none;
            }
        }

        // nested child element
        .child-container {
            width: 50%;
        }
    }

Try to keep nested rules to a maximum of 3 levels deep.

**Using Bootstrap's Grid System**

Wanderable's site is fully responsive, largely relying on [Bootstrap's responsive grid](http://getbootstrap.com/css/#grid) and responsive utility classes. 

Here are some examples of responsive columns using Bootstrap:

    <div class="col-xs-12 col-sm-6 col-md-4 col-lg-3"></div>
    <div class="col-xs-12 col-sm-3"></div>

Refer to Bootstrap's documentation for a detailed explanation of these helper classes.

**Responsive Philosophy**

Unlike Bootstrap, our responsive development is not mobile-first. This is somewhat due to an informal design process. 

Most existing Less code was written desktop-first (768px - 991px). This was done to support browsers (IE8) with no media query features. However, since dropping IE8 support in March 2014, writing desktop-first is no longer a priority.

**Media Query Helpers**

Wanderable has additional media query helper variables built on top of Bootstrap's variables. These variables were added because Bootstrap's media queries felt clunky, repetitive and unintuitive.

Here are the custom Wanderable variables in pseudocode: 

    @screen-phone-max: 767px;
    @screen-tablet-max: 991px;
    @screen-desktop-max: 1199px;

    @phone-v: (max-width: 319px); // phone-vertical
    @phone-h: (min-width: 320px) and (max-width: 479px); // phone-horizontal
    @phone: (max-width: 479px);
    @tablet: (min-width: 480px) and (max-width: 767px); 
    @desktop: (min-width: 768px) and (max-width: 991px); 
    @desktop-lg: (min-width: 992px); 

    @lt-tablet: (max-width: 479px); 
    @lt-desktop: (max-width: 767px); 
    @lt-desktop-lg: (max-width: 991px); 

    @desktop-only: (min-width: 768px); 

These variables add a layer of semantic meaning and reduces repetition in the responsive code. Here is what the same media query looks like using Bootstrap's variables versus Wanderable's variables.

    // Bootstrap's Media Query Variables
    @media (min-width: @screen-xs) and (max-width: @screen-sm-max) {
        // responsive code here
    }

    // Wanderable's Media Query Variables
    @media @tablet {
        // responsive code here
    }


**Wanderable's Responsive Utilities**

On top of responsive variables, there are also additional responsive helper classes defined in `global/_responsive-utilities.less`. These classes are mostly used to handle general `@phone-v` cases, which bootstrap does not support.

## Responsive Images with Cloudinary 

In `global/_responsive-img-mixins.less`, you will find a number of mixins written to work with Cloudinary's image processing API. 

These mixins allow us to pass in certain parameters like `cloudinary_id`, `width`, `height`, `opacity`, etc. into a Less mixin. The mixin will compile these params into image URLs that correspond to the current viewport size. 

The mixins save us the trouble of writing media queries that look like this:

    .hero-unit {
        background-image: url('image-url.png');
        background-size: cover;
        background-repeat: no-repeat;

        @media @phone { background-image: url('image-url-phone.png'); }
        @media @tablet { background-image: url('image-url-tablet.png'); }
    }

*The above is very tedious.*

## Additional Notes 

### Wanderable's Color Palette

Wanderable's color palette as defined in `_variables.less`:

> TODO add image here

### Using the smooth-scrolling plugin

The smooth-scrolling plugin enables smooth-scrolling to any anchor link on the same page. It is located in `global/smooth-scroll.js`.

*Usage*
Add the data attribute `data-smoothscroll='true'` to any anchor link. 

    <a href="#signup-form" data-smoothscroll="true">Go to signup</a>

*Voila!*

### Parallax image plugin

The parallax scrolling plugin enables a parallax effect on any element with a background image. It is located in `global/parallax-scrolling.js`.

*Usage*
Add the class `.parallax-scroll` to any element with a background image.
This class adds both CSS styles and JS effects to the element.

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

**JS file names**

The Javascript files currently use `_` as delimiters, and good practice is to name the JS file the same thing as the view file. This gets confusing if there are two views in different directories with the same name (_form, etc.) so use your discretion and grep accordingly.

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
