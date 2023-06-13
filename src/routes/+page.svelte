<script>
    import Cropper from "../lib/Cropper.svelte";
    import Cropped from "../lib/Cropped.svelte";

    let allCrops = [];
    let filename;

    let innerWidth;
</script>

<svelte:window bind:innerWidth={innerWidth}/>

<!-- App can only be used on tablet size and up. -->
<div class="main-body" class:hide="{innerWidth < 767}">
    <Cropper cwidth=500 cheight=750 bind:allCrops={allCrops} bind:filename={filename}/>
    <Cropped bind:allCrops={allCrops} bind:filename={filename}/>
</div>
<div class="section main-body" class:hide="{innerWidth > 767}">
    <p>Seems like the screen is too small!</p>
</div>

<style>
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
</style>