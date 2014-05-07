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

    The asset pipeline provides a framework to concatenate and minify or compress JavaScript and CSS assets. It also adds the ability to write these assets in other languages and pre-processors such as CoffeeScript, Sass and ERB.

- Concatenation and minifying through Sprockets
- Less requests
- Cached 
- Using manifests and bundles
- cloudinary
- caching responsive
The biggest reason for using the Asset Pipeline is concatenation and minifying of all assets on the site. 
Fewer requests automatically mean 

    E.g.

The Asset Pipeline allows us to concatenate all specified LESS or Coffee files into master CSS and JS files, which load in a single request upon initial site visit. These files are then cached in the user's browser, and cost significantly less load time throughout the rest of a user's browsing experience or on return visits. 

Here is an example using the Networks tab in the [Chrome Developer Tools](https://developers.google.com/chrome-developer-tools/docs/network)

    E.g. Take a look at the networks panel in the Chrome Devtools on initial load

    Now take a look at it on page reload
    (explain this)

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
