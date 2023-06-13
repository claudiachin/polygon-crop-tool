<script>
    import Cropper from "../lib/Cropper.svelte";
    import Cropped from "../lib/Cropped.svelte";

    let allCrops = [];
    let filename;
</script>

<Cropper cwidth=500 cheight=750 bind:allCrops={allCrops} bind:filename={filename}/>
<Cropped bind:allCrops={allCrops} bind:filename={filename}/>