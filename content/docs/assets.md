# Importing Assets

Images, stylesheets and other assets are an important part of any web application. These things are integrated right into imba without any need for external bundlers like webpack or rollup. 

## Importing stylesheets

Even though you can style all of your app using the imba css syntax you might want to import external stylesheets as well. Import these just like you would import any other file.

```imba app.imba
import './assets/normalize.css'

tag App
	# ...
```

In the example above, the content of the normalize.css file will automagically be imported and bundled alongside your other styles.

You can also import the css generated by imba via a non-standard `src` attribute on the `style` element. This is especially useful for ssr applications where you don't want to ship your rendering code to the client, but obviously want the styles:

```imba server.imba
import {App} from './app'
app.get(/.*/) do(req,res)
	res.send String <html>
		<head>
			<title> "Application"
			<style src='./app'>
		<body> <App>
```

```imba app.imba
global css
	body bg:gray2 # ...

export tag App
	<self>
		<div[fw:bold]> "Welcome"
```

## Importing images

Any relative url reference you have in your styles will be extracted, hashed, and included with your bundled code.

```imba app.imba
tag App
	<self> <div[ bg:url(./logo.png) ]>
```

Also, relative src attributes on `<img>` elements will automatically be handled as assets

```imba app.imba
tag App
	<self> <img src='./logo.png'>
```

If you want to reference and use these images from your code as well, you can import them:

```imba app.imba
import logo from './logo.png'

tag App
	<self> <img src=logo>
```

These imported assets even have valuable metadata added to them:

```imba app.imba
import logo from './logo.png'

logo.url # the unique hashed url for this image
logo.size # size of assets - in bytes
logo.hash # unique hash of the contents of the asset
logo.body # the actual contents of the asset - only available on server
logo.width # width of the imported image
logo.height # height of the imported image

tag App
	<self> <img src=logo width=logo.width>
```

You can also use the dynamic import syntax for assets:

```imba app.imba
const states = {
	open: import('./icons/alert-circle.svg')
	closed: import('./icons/check-circle.svg')
	archived: import('./icons/package.svg')
}

<div> for issue in issues
	<div>
		<img.icon src=states[issue.state]>
		<span> issue.title
```

You can even interpolate these dynamic asset imports in css:

```imba app.imba
const states = {
	open: import('./icons/alert-circle.svg')
	closed: import('./icons/check-circle.svg')
	archived: import('./icons/package.svg')
}

<div> for issue in issues
	<div>
		<.icon[ bg:{states[issue.state]} ]>
		<.title> issue.title
```

## Importing svgs

SVGs can be imported and used just like other image assets described in the previous section. It does include some nice additional things though. You can use SVG images via css and the img-tag, but they really shine when you inline them in your html. Ie, if you want to use SVG images for icons you have to inline them to be able to change their color, stroke-width etc.

So, Imba adds a non-standard `src` attribute to `svg` elements, that magically inlines the actual content of the svg directly in your html.

```imba app.imba
<div>
	# the package.svg will now be inlined in this tag
	<svg src='./icons/package.svg'>
```

This also allows us to style the contents directly:

```imba app.imba
<div>
	# change color on hover
	<svg[c:gray6 @hover:blue7] src='./icons/package.svg'>
	# override stroke-width 
	<svg[stroke-width:4px] src='./icons/check.svg'>
```

## Importing scripts

Imba will analyze the `src` attribute of `script` elements and automatically package these files for you. So a typical project with server *and* client code will follow this pattern:

```imba server.imba
# some express server ...
app.on(/.*/) do(req,res)
	res.send String <html>
		<head>
			<title> "Gobias Industries"
		<body>
			<script type="module" src="./client">
```

```imba client.imba
# some client code here
```

When you run `imba server.imba` in the above example, imba will discover the reference to client.imba and create a bundle for the `client.imba` entrypoint. Using [esbuild](https://esbuild.github.io/) in the background, bundling is so fast that it literally happens every time you start your server – *there is no need for a separate build step*.

## Importing workers

Imba aims for zero-configuration bundling even with large projects. You can import scripts as separate assets via a special import syntax.

```imba app.imba

const script = import.worker './compiler'
# script is now an asset reference to the bundled compiler entrypoint.
script.url # reference to the unique/hashed url of this script
script.body # node only - actual contents of the bundle

const worker = new Worker(script.url)
```

To showcase how flexible and useful this is – on imba.io we are using a serviceworker to run examples. Serviceworkers need to be served from the directory / scope you want them to control, and their url need to be static. The asset urls generated by imba are usually hashed *and* put inside a special `__assets__` folder:

```imba app.imba
const sw = import.worker './sw/service'
sw.url # /__assets__/sw/worker.JNA7FI8P.js
```
This url changes whenever the content changes and is nested inside an `/__assets__/...` prefix, both of which can be challenging for a serviceworker. So, our solution was to create a static route in our server, and just serve the asset directly:

```imba app.imba
app.get('/sw.js') do(req,res)
	const asset = import.worker('./sw/service')
	res.send asset.body
```

## Custom imports

The `import.worker` syntax shown in the previous section is not a special syntax just for workers. In fact, the `import.anything(...)` is a general way to import something using a specific configuration. You can create such configurations in your `imbaconfig.json`. Imba comes with a few presets:

```imba presets.imba
export const presets = 
	node:
		platform: 'node'
		format: 'cjs'
		sourcemap: true
		target: ['node12.19.0']
		external: ['dependencies','!imba']
		
	web:
		platform: 'browser'
		target: ['es2020','chrome58','firefox57','safari11','edge16']
		sourcemap: true
		format: 'esm'
		
	iife:
		extends: 'web'
		format: 'iife'
		
	client:
		extends: 'web'
		splitting: true
		
	worker:
		format: 'esm'
		platform: 'worker'
		splitting: false

```

Most of the properties map directly to [esbuild options](https://esbuild.github.io/api/#simple-options), with some additions for imba specific things. Configuration options will be explained in more detail before final 2.0 release. In most projects you will not need to think about or tweak these configs.

## Future plans

### Glob imports

We are working on smart glob imports for a future release.

```imba app.imba
import icons from './feather/*.svg'

# contains all matching assets:
icons.check
icons.circle
...
```

This will also work directly in src paths, like:

```imba app.imba
<div> for issue in issues
	<div>
		<svg src="./icons/{issue.state}.svg">
		<div.title> issue.title
```

### Importing wasm

Will be part of a future release.
