# Polygon Cropping Tool
Cropping tool made for Svelte using HTML5 Canvas.

## Features
- Upload file
- Polygon cropping
- Rectangle cropping
- Undo, Redo, Reset
- Manage crops
- Save crops (as json)

## How to use
1. Install 
```bash
npm i polygon-crop-tool
```

2. Add to your project
```svelte
// routes/+page.svelte
<script>
    import {Cropper, Cropped} from 'polygon-crop-tool';

    // variables to download all the cropped images
    let allCrops = [];
    let filename;
</script>

<Cropper 
    colour="#ED3996" // colour must be in HEX code. determines the colour of the lines drawn on the canvas.
    cwidth=500 // determines the width of the canvas
    cheight=750 // determines the height of the canvas (before an image is loaded)
    bind:allCrops={allCrops} 
    bind:filename={filename}
/>
<Cropped bind:allCrops={allCrops} bind:filename={filename}/>
```

3. Done!

## Additional Info
Styling can be done to make <code><Cropper /></code> and <code><Cropped /></code> look better. 
```svelte
<script>
    import {Cropper, Cropped} from 'polygon-crop-tool';

    // variables to download all the cropped images
    let allCrops = [];
    let filename;

    // media query
    let innerWidth;
</script>
<svelte:window bind:innerWidth={innerWidth}/>

<!-- App can only be used on tablet size and up. -->
<div class="main-body" class:hide="{innerWidth < 767}">
    <Cropper colour="#ED3996" cwidth=500 cheight=750 bind:allCrops={allCrops} bind:filename={filename}/>
    <Cropped bind:allCrops={allCrops} bind:filename={filename}/>
</div>
<div class:hide="{innerWidth > 767}">
    <p>Seems like the screen is too small!</p>
</div>

<style>
    .main-body {
        background: #FFFFFF;
        display: flex;
        flex-wrap: wrap;
        justify-content: center;
        gap: 24px;
    }

    .hide {
        display: none !important;
    }

    @media screen and (min-width: 1025px) {
        :global(.cropper-wrapper) {
            justify-content: end !important;
        }
    }
</style>
```

## Upcoming features
- Circular cropping
- Save crops (as images)

<!-- # create-svelte

Everything you need to build a Svelte library, powered by [`create-svelte`](https://github.com/sveltejs/kit/tree/master/packages/create-svelte).

Read more about creating a library [in the docs](https://kit.svelte.dev/docs/packaging).

## Creating a project

If you're seeing this, you've probably already done this step. Congrats!

```bash
# create a new project in the current directory
npm create svelte@latest

# create a new project in my-app
npm create svelte@latest my-app
```

## Developing

Once you've created a project and installed dependencies with `npm install` (or `pnpm install` or `yarn`), start a development server:

```bash
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open
```

Everything inside `src/lib` is part of your library, everything inside `src/routes` can be used as a showcase or preview app.

## Building

To build your library:

```bash
npm run package
```

To create a production version of your showcase app:

```bash
npm run build
```

You can preview the production build with `npm run preview`.

> To deploy your app, you may need to install an [adapter](https://kit.svelte.dev/docs/adapters) for your target environment.

## Publishing

Go into the `package.json` and give your package the desired name through the `"name"` option. Also consider adding a `"license"` field and point it to a `LICENSE` file which you can create from a template (one popular option is the [MIT license](https://opensource.org/license/mit/)).

To publish your library to [npm](https://www.npmjs.com):

```bash
npm publish
``` -->
