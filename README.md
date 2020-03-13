reveal.js-docker
===

Docker images providing easier to use, opinionated reveal.js web apps - web-based slides/presentations.

Based on [cloudogu/continuous-delivery-slides](https://github.com/cloudogu/continuous-delivery-slides)

# How to use

## Simplest start 

Ships a default presentation:

```bash
docker run --rm -p 8080:8080 cloudogu/reveal.js
```

## Ship your own slides

* Mount md slides to `/docs/slides`
* Optionally mount folders to web folder, e.g. like so:  
 `-v  $(pwd)/images:/reveal/images`
* Mount folder containing HTML Fragment-files to `/resources`
  * `slides.html` -> Pick the slides from `docs/slides` ([example](scripts/test/slides.html))
  * `additional.js` - Script executed before initializing reveal.js
  * `body-end.html` - `html` injected at the end of `body
  * `footer.html` - rendered at the footer (lower left corner) for now only works with cloudogu Themes
* Optional Env vars: 
  * `TITLE`
  * `THEME_CSS`
     * `css/cloudogu.css`
     * `css/cloudogu-black.css`
  * `SHOW_NOTES_FOR_PRINTING` - print speaker notes - defaults to `false`.
  * `ADDITIONAL_REVEAL_OPTIONS` - additional reveal.js options, e.g. pdfSeparateFragments : false 
  * `ADDITIONAL_DEPENDENCIES` - additional reveal.js dependencies, e.g. plugins  
     e.g. `-e ADDITIONAL_DEPENDENCIES="{ src: 'plugin/tagcloud/tagcloud.js', async: true }" `  
     Note that these files have to be mounted to the /reveal folder, e.g. here `-v $(pwd)/plugin/tagcloud:/reveal/plugin/tagcloud`
* Start Container

```bash
# Development mode (with live reloading)
docker run --rm \
    -v $(pwd)/docs/slides:/reveal/docs/slides \
    -v $(pwd)/scripts/test:/resources \
    -e TITLE='my Title' -e THEME_CSS='css/cloudogu-black.css' \
    -p 8000:8000 -p 35729:35729 \
    cloudogu/reveal.js:dev

# Production Mode (smaller, more secure, just a static HTML site served by NGINX)
docker run --rm \
    -v $(pwd)/docs/slides:/reveal/docs/slides \
    -v $(pwd)/scripts/test:/resources \
    -e TITLE='my Title' -e THEME_CSS='css/cloudogu-black.css' \
    -e ADDITIONAL_REVEAL_OPTIONS='pdfSeparateFragments: false' \
    -e SHOW_NOTES_FOR_PRINTING='false' \
    -p 8080:8080 \
    cloudogu/reveal.js
```

## Index.html Template and params:

An overview where the env vars and  HTML Fragment are injected:

```html
<!doctype html>
<html>
<head>
    <title>${TITLE}</title>
    
    <link rel="stylesheet" href="${THEME_CSS-css/cloudogu.css}">
</head>
<body>
    <div class="reveal">
        <!-- optional: footer.html -->
        <div class="slides">
        <!-- slides.html-->
        </div>
    </div>
    const showNotesForPrinting = ${SHOW_NOTES_FOR_PRINTING};
    <!-- Optional: <script>additional.js</script>-  -->
    <script>
    // ... Initializes reveal.js
        dependencies: [
                         // ...,
                         ${ADDITIONAL_DEPENDENCIES}
                       ]
        ${ADDITIONAL_REVEAL_OPTIONS}
    </script>
    
    <!-- optional: body-end.html -->
</body>
</html>
```

# Development

Build Docker Images, from repo root

```bash
docker build -t prod .
docker build -t dev --build-arg ENV=dev .
```

Note: If only one build is required, buildkit would be more efficient. However, prod is failing with buildkit.
Try it with ` export DOCKER_BUILDKIT=1
See https://github.com/moby/moby/issues/735

## Tests

Run tests locally

```bash
# For now manually
cd scripts/test
../src/templateIndexHtml
```
