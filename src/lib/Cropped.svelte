<script>
    import 'material-icons/iconfont/outlined.css';

	export let allCrops;
	export let filename;

	function save() {
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

<div class="section cropped-images-wrapper">
	<div>
		<h6>Images cropped:</h6>
		<div id="cropped-imgs" />
	</div>
	<button on:click={save} disabled={allCrops.length == 0} id="save" title="Save"
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
		border-radius: 1000px;
		padding: 8px 32px;
		border: 2px solid #ed3996;
		width: max-content;
		background: transparent;
	}

	.cropped-images-wrapper #save:hover {
		border: 2px solid #d51477;
	}

	.cropped-images-wrapper span {
		font-size: 16px;
		color: #000000;
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
