<script>
    import 'material-icons/iconfont/outlined.css';

	export let allCrops;
	export let filename;
	export let colours;

	function save() {
		console.log(allCrops);
		var downloadClick = document.createElement('a');
		downloadClick.setAttribute(
			'href',
			'data:text/plain;charset=utf-8,' + encodeURIComponent(JSON.stringify(allCrops, null, '\t'))
		);
		downloadClick.setAttribute('download', filename + '_crops.json');

		downloadClick.style.display = 'none';
		document.body.appendChild(downloadClick);

		downloadClick.click();

		document.body.removeChild(downloadClick);
	}
</script>

<div class="section cropped-images-wrapper" 
	style="--primary-color: {colours.primary}; 
	--secondary-color: {colours.secondary};"
>
	<div>
		<h6>Images cropped:</h6>
		<div id="cropped-imgs" />
	</div>
	<button on:click={save} disabled={allCrops.length <= 1} id="save" title="Save"
		><span class="material-icons-outlined">save</span>
		<p>Save</p></button
	>
</div>

<style>
	.cropped-images-wrapper {
		flex: 1 0 400px;

		display: flex;
		flex-direction: column;
		justify-content: space-between;
	}

	.cropped-images-wrapper #save {
		display: flex;
		gap: 8px;
		justify-content: center;
		align-items: center;
		width: max-content;
	}

	.cropped-images-wrapper span {
		font-size: 16px;
		color: #404040;
	}

	#cropped-imgs {
		display: flex;
		flex-direction: column;
		gap: 8px;
	}

	button:disabled {
		opacity: 0.5;
	}
</style>
