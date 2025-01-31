
# adonis5-swagger
> Swagger, AdonisJS, SwaggerUI

[![typescript-image]][typescript-url] [![npm-image]][npm-url] [![license-image]][license-url]

Create API documentation easily in Adonis 5 using [Swagger](https://swagger.io/specification/)

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
## Table of contents

- [Installation](#installation)
- [Sample Usage](#sample-usage)
- [Best usage](#best-usage)
- [Custom UI](#custom-ui)
- [Build swagger spec file](#build-swagger-spec-file)
- [Swagger modes](#swagger-modes)
- [Production using](#production-using)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Installation
```bash
npm i --save adonis5-swagger
```
Compile your code:
```bash
node ace serve --watch
```
Connect all dependences:
```bash
node ace invoke adonis5-swagger
```
* For other configuration, please update the `config/swagger.ts`.

# Sample Usage
* Add new route:
  ```js
  Route.get('/api/hello', 'TestController.hello')
  ```

* Create `TestController` using `node ace make:controller Test` command:
  ```js
  import { HttpContextContract } from '@ioc:Adonis/Core/HttpContext'
  
  export default class TestController {
    /**
    * @swagger
    * /api/hello:
    *   get:
    *     tags:
    *       - Test
    *     summary: Sample API
    *     parameters:
    *       - name: name
    *         description: Name of the user
    *         in: query
    *         required: false
    *         type: string
    *     responses:
    *       200:
    *         description: Send hello message
    *         example:
    *           message: Hello Guess
    */
    public async hello({ request, response }: HttpContextContract) {
      const name = request.input('name', 'Guess')
      return response.send({ message: 'Hello ' + name })
    }
  }
  ```

* You can also define the schema in the Models:
  ```js
  import { BaseModel } from '@ioc:Adonis/Lucid/Orm'
  
  /** 
  *  @swagger
  *  definitions:
  *    User:
  *      type: object
  *      properties:
  *        id:
  *          type: uint
  *        username:
  *          type: string
  *        email:
  *          type: string
  *        password:
  *          type: string
  *      required:
  *        - username
  *        - email
  *        - password
  */
  export default class User extends BaseModel {
  }
  ```

* Or create a separate file containing documentation from the APIs in either TS or YAML formats, sample structure:
  ```bash
  project
  ├── app
  ├── config 
  ├── docs
  │   ├── controllers
  │   │   ├── **/*.ts
  │   │   ├── **/*.yml
  │   └── models
  │       ├── **/*.ts
  │       ├── **/*.yml
  ```
# Best usage
* Create files into docs/swagger, for example docs/swagger/auth.yml may contains:

```YAML
/api/auth/login:
  post:
    tags:
      - Auth
    security: []
    description: Login
    parameters:
      - name: credentials
        in:  body
        required: true
        schema:
          properties:
            phone:
              type: string
              example: '1234567890'
              required: true
    produces:
      - application/json
    responses:
      200:
        description: Success
```
* You can change default settings in config/swagger.ts
* For other sample in YAML and JS format, please refer to this [link](/sample).

Open http://localhost:3333/docs in your browser
For detail usage, please check the Swagger specification in this [SwaggerSpec](https://swagger.io/specification/)

# Custom UI
For using custom ui you should use own build of swagger ui. Current package contains only preconfigured and already built ui bundle. For example, you can use [Adonis Edge](https://preview.adonisjs.com/packages/edge) for rendering ui with custom params.

First, install edge:
```bash
npm i @adonisjs/view
```
Once installed, run the following ace command to setup the package.
```bash
node ace invoke @adonisjs/view
```
Generate new template file using Adonis generator:
```bash
node ace make:view swagger
```

And then add route for custom UI:
```js
Route.get('/', async ({ view }) => {
	const specUrl = 'your spec url'
	return view.render('swagger', { specUrl })
})
```

Your template should have similar content:
```html
<!DOCTYPE html>
<head>
	<script src="//unpkg.com/swagger-ui-dist@3/swagger-ui-standalone-preset.js"></script>
	<script src="//unpkg.com/swagger-ui-dist@3/swagger-ui-bundle.js"></script>
	<link rel="stylesheet" href="//unpkg.com/swagger-ui-dist@3/swagger-ui.css"/>
	<script>
		window.onload = () => {
			let ui = SwaggerUIBundle({
				url: "{{ specUrl }}",
				dom_id: "#swagger-ui",
				presets: [
					SwaggerUIBundle.presets.apis,
					SwaggerUIBundle.SwaggerUIStandalonePreset
				],
				plugins: [
					SwaggerUIBundle.plugins.DownloadUrl
				],
			})

			window.ui = ui
		}
	</script>
</head>
<body>
	<div id="swagger-ui"></div>
</body>
</html>
```
It is the simplest way for using custom swagger ui, but of course you could use Webpack or another bundler tool for bundling your pre configured Swagger ui.
# Build swagger spec file

You can build swagger spec file the next way:

Set `specFilePath` option to your swagger config:
```js
const swaggerConfig = {
	specFilePath: 'docs/swagger.json'
}
```
And then run adonis command:
```bash
node ace swagger:generate
```
Generated file will be written to by path configured in config.

# Swagger modes
This package support two modes:
- PRODUCTION
- RUNTIME

By default RUNTIME mode enabled. When RUNTIME mode enabled package rebuild swagger spec file on each request. When you use PRODUCTION you should build your swagger spec once and then package will be respond this file content on each request.

# Production using
For using swagger in production you should make some preparations:
-	Setup swagger config:
```js
const swaggerConfig = { 
	mode: process.env.NODE_ENV === 'production' ? 'PRODUCTION' : 'RUNTIME',
	specFilePath: 'docs/swagger.json'
}
```
- Add post hook for npm build script to your `package.json`:
```json
{
	"scripts": {
		"build": "npm run compile",
		"postbuild": "node ace swagger:generate && cp -a docs/ build/docs"
	}
}
```
- Deliver your source code to production server.


[typescript-image]: https://img.shields.io/badge/Typescript-294E80.svg?style=for-the-badge&logo=typescript
[typescript-url]:  "typescript"

[npm-image]: https://img.shields.io/npm/v/adonis5-swagger.svg?style=for-the-badge&logo=npm
[npm-url]: https://npmjs.org/package/adonis5-swagger "npm"

[license-image]: https://img.shields.io/npm/l/adonis5-swagger?color=blueviolet&style=for-the-badge
[license-url]: LICENSE.md "license"
