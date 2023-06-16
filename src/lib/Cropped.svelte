<script>
    import 'material-icons/iconfont/outlined.css';

	export let allCrops;
	export let filename;
	export let colours;

	function save(item, ext) {
		var downloadClick = document.createElement('a');

		downloadClick.setAttribute('href',item);
		downloadClick.setAttribute('download', filename + '_crops' + ext);

		downloadClick.style.display = 'none';
		document.body.appendChild(downloadClick);

		downloadClick.click();

		document.body.removeChild(downloadClick);
	}

	function saveImgs() {
		for (const item of allCrops.slice(1)) {
			save(item.base64, '.png');
		}
	}

	function saveJson() {
		save('data:text/plain;charset=utf-8,'+encodeURIComponent(JSON.stringify(allCrops, null, '\t')), '.json');
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
	<div class="save-buttons">
		<button on:click={saveJson} disabled={allCrops.length <= 1} id="save" title="Save">
			<span class="material-icons-outlined">save</span>
			<p>Save as .json</p>
		</button>
		<button on:click={saveImgs} disabled={allCrops.length <= 1} id="save" title="Save">
			<span class="material-icons-outlined">save</span>
			<p>Save as .png</p>
		</button>
	</div>
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

	.save-buttons {
		display: flex;
		gap: 8px;
	}
</style>
