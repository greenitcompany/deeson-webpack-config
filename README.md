# Deeson frontend tooling

This project pulls together our frontend toolkit into one (yarn|npm)able module.

## How and Why

Deeson wants to develop frontend reasonably independently from any CMS, such as Drupal, and practice Component Driven Frontend Development.

A component is a collection of HTML, CSS and JS that goes together to form some display element. Consider this against a more traditional approach where HTML, CSS and JS are stored in separate global files, for example, in a global style.css and app.js files. By grouping code together into components, we separate our application by the domain language, rather than arbitrarily by file extension. This isolates components and makes them easier to develop and maintain.

The output of our frontend development will include a style guide which will render each of our components without Drupal. 

Style guides are useful as they demonstrate the components independently of the specific implementation. This allows more rapid frontend development as frontend developers can work without having to worry about how the backend will integrate.

Typically however, these style guides quickly get out of sync with the applications they were developed for. The Drupal developer must later integrate the HTML produced into the finished site which has meant copying and pasting code out of the style guide templates and into Drupal templates. At this point we have duplication of code and the ability to maintain a strict style guide is lost. When the style guide is updated, all Drupal templates affected must be identified and updated as well. 

Our approach makes the style guide a living style guide. We use TWIG as the HTML template for our components meaning that our twig templates inside our frontend component code are the exact same ones that Drupal will be using within the theme. Frontend developers can make changes to it knowing that those changes will flow through into the application without need for a second step.  This does mean that Drupal developers often have to modify the templates to work exactly as Drupal expects but that’s fine and keeps the templates in sync with both frontend and backend.

### Prerequisites

You will need to have the following tools installed locally. If you are on a Mac then we recommend installing them using [Homebrew](https://brew.sh/) where available

* [node](https://nodejs.org)
* [yarn](https://yarnpkg.com)
* [php](https://php.net)

For Drupal 8 integration you will also need:

* [composer](https://getcomposer.org/)

If you are using with Drupal 8 then we recommend starting development with our [Drupal 8 quickstart recipe](https://github.com/teamdeeson/d8-quickstart). This receipe comes with our frontend preconfigured and ready to go in the `src/frontend` folder.

### Installing

Not required if you are starting with our [Drupal 8 quickstart recipe](https://github.com/teamdeeson/d8-quickstart). Jump to the next section on running your frontend locally.

1. Make yourself a fresh directory and `yarn init` inside it. Follow all the normal npmish instructions.
2. `yarn add https://github.com/teamdeeson/deeson-webpack-config`
3. If you are going to need twig templates (drupal 8 or any number of other setups) then you should `composer require twig/twig` too
4. Open up your `package.json` and add the following to the `scripts` section
   ```javascript
   "start": "./node_modules/.bin/frontend-serve",
   ```
5. Make yourself a `webpack.config.js` file that looks like this
   ```javascript
   const config = require('deeson-webpack-config-starter');

   // this should be the real asset path in drupal
   // so something like /sites/all/themes/blah_blah/assets
   config.output.publicPath = '/assets/';

   // this is how you get auto reload happening
   // for your pages without adding them to ./src/app.js
   config.entry.pages = './pages/index.js';

   module.exports = config;
   ```

We just need a couple more files...

1. An entry point for webpack, our default is `src/app.js`
   ```javascript
   console.log('hello')
   ```
2. An index page that we can add our components to so we can make them awesome, `pages/index.php` or `pages/index.twig`.
   ```html
   <!doctype html>
   <html>
     <head>
       <link rel="stylesheet" href="{{ directory }}/assets/app.css">
       <script type="text/javascript" src="{{ directory }}/assets/app.js"></script>
     </head>
     <body>
       <h1>index.php</h1>
     </body>
   </html>
   ```
3. Another entry point for webpack so it can watch our development pages (this is a bit of a work in progress to give us live reload), our default is `pages/index.js`.
   ```javascript
   import './index.php'
   ```

### Running

If you are in the root of your source code then the following commands will start the process of running your development locally. If you are using our Drupal 8 quickstart, you'll need to be in the `src/frontend` folder first:

`yarn start` and visit `https://localhost:8080/pages/index.php`.

Now get creating.

### File naming and indexes

You'll want to make sure that your pages all go in to a `/pages` dir (and compiled assets go into `/assets`). 
Files in `/pages` need to have a `.twig.html` or `.php.html` extension if you want to 
be able to make a static version of your site. Our router is set up to handle 
these extensions just fine, but you may need to configure them in your IDE if 
you want syntax highlighting.

Files elsewhere (such as under `/src`) should not add the `.html` suffix.

The router also provides basic autoindexing support by redirecting / to 
/index.twig.html automatically but static hosting environments are unlikey to 
support that behaviour so please use full urls in hyperlinks.   

### Drupal Integration

Our webpack [plugin](https://webpack.js.org/concepts/plugins/) keeps track of all our template related files during build and outputs a bit of php at the end that lets us do the following.

#### Seven and Eight

Inside of a themes `template.php` for 7 and `<theme>.theme` for 8 (and assuming you output assets to /assets/)
```php
require_once __DIR__ . '/assets/component-templates.php';

function <yourtheme>_theme() {
  // for 7
  return deeson_tpl_component_templates();
  // or for 8
  // return deeson_twig_component_templates();
}
```

This sets up each of your components as a template callable in the same way you would anywhere else.
```php
// render array
function some_sort_of_preprocess(&$vars) {
  $vars['content'][] = [
    '#theme' => 'yourcomponent',
    '#content' => ['arguments' => 'in here'],
  ];
}

// drupal 7 theme function
<?php print theme('yourcomponent', ['content'=>['arguments'=>'in here']]); ?>
```

#### Drupal 8 

We make use of the [components](https://www.drupal.org/project/components) module to set us up with a namespace to reference our components from. So once you have it installed stick something like the following in `<yourtheme>.info.yml`
```yml
component-libraries:
  components:
    paths:
      - assets/components
```

This will allow you to reference components directly from templates
```twig
{% include '@components/mycomponent/mycomponent.html.twig' with {content: {arguments: 'in here'}} %}
```

### Referencing images from templates and styles
For the examples below, assume a component tree like this one:
```
src/
├── app.js
└── components
    └── someComponent
        ├── index.js
        ├── index.scss
        ├── some-component.html.twig
        ├── large-image.jpg
        └── small-image.svg
```

#### …from SASS
Referencing images from SASS is handled automatically. You can reference images from the relative path to your SASS file.
So in the above example, from index.scss you could reference `small-image.svg` with a rule like this one:
 
```
div {
  background: url('./small-image.svg');
}
```

Small images (at time of writing the actual limit is  below 1000 bytes) will be inlined as a data url.
Larger files are emitted to the assets directory with the url reference being maintained.
 
#### …from a template
Referencing images from the template is a little more complicated because Webpack cannot interpret the twig and php files we create.
To reference an image from a template you'll have to do two things:

1. `import` the file. So in `index.js`, you would have a line that looked like `import "./large-image.jpg"`.
  
2. In the template itself, use an absolute path, prefixed with `{{ base_path }}{{ directory }}/assets/`. 
So to reference `large-image.jpg` from above you would have something like this:
`<img src="{{ base_path }}{{ directory }}/assets/components/someComponent/large-image.jpg" />`

Both Drupal 7 and 8 can provide these variables, so for 7 you can replace with 
`<?php print $base_path . $directory; ?>`.

### Static builds
You can create a static export of your application for deployment to pretty much 
any webserver using the `deeson-static-build` script.

Call it like this:

`./node_modules/.bin/deeson-static-build /dir/to/output /base/url/prefix`

Where `/dir/to/output` is the path where you'd like the compiled assets placed.</br>
And `/base/url/prefix` is the path from which the site will be served.

E.g. if you were hosting on https://static.example.com/our-site, then your 
command might look like this:
 
`./node_modules/.bin/deeson-static-build /var/www/vhosts/static.example.com/our-site /our-site`

The second URL prefix parameter is injected into your templates as `{{ directory }}`.


## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details

