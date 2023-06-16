<script>
    import Cropper from "../lib/Cropper.svelte";
    import Cropped from "../lib/Cropped.svelte";

    let allCrops = [];
    let filename;

    let colours = {
        primary: '#FF6B6B', // cropping lines + fill
        secondary: '#4ECDC4', // axis lines
        tooltip: '#121212',
        tooltip_text: '#FFFFFF',
    }

    let innerWidth;
</script>

<svelte:window bind:innerWidth={innerWidth}/>

<!-- App can only be used on tablet size and up. -->
<div class="section main-body" class:hide="{innerWidth < 767}">
    <Cropper cwidth=500 cheight=750 colours={colours} bind:allCrops={allCrops} bind:filename={filename}/>
    <Cropped colours={colours} bind:allCrops={allCrops} bind:filename={filename}/>
</div>
<div class="section main-body" class:hide="{innerWidth > 767}">
    <p>Seems like the screen is too small!</p>
</div>

<style lang="scss">
    @import url('https://fonts.googleapis.com/css2?family=Akshar:wght@300;700&display=swap');

    $black: #121212;
    $white: #FFFFFF;
    $primary: #FF6B6B;
    $secondary: #4ECDC4;

    :global(html) {
        scroll-behavior: smooth;
    }

    :global(body) {
        margin: 0;
    }

    :global(h6, p) {
        font-family: 'Akshar', sans-serif;
        text-align: start;
        margin: 0;
        color: #121212;
    }

    :global(h6) {
        font-size: 20px;
        font-weight: bold;
        font-style: normal;
    }

    :global(p) {
        font-size: 16px;
        font-weight: normal;
        font-style: normal;
        margin: 4px 0;
        white-space: pre-line;
    }

    :global(a) {
        text-decoration: none;
    }

    :global(.section) {
        padding: 32px;
    }

    .main-body {
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

    :global(.upload-button) {
        background: transparent;

        :global(span) {
            color: $black;
        }
    } 

    :global(.cropped-img-div) {
        border: solid 2px $secondary !important;
    }

    :global(.save-buttons > button) {
        border-radius: 1000px;
        padding: 8px 32px;
        border: 2px solid $primary;
        background: transparent;
    }

    :global(.save-buttons > button:hover) {
        border-radius: 1000px;
        padding: 8px 32px;
        border: 2px solid $secondary;
        background: transparent;
    }
</style>