<script>
	// Code heavily inspired by this: https://stackoverflow.com/questions/27213413/canvas-cropping-images-in-different-shapes

    import 'material-icons/iconfont/outlined.css';
	import { onMount } from 'svelte';

	// variables for uploading image
	let input;
	let uploadBtn;

	// canvas-related variables
	let canvas;
	let ctx;
	export let cwidth;
	export let cheight;

	// image to be cropped
	let img;

	// div to store cropped images
	let croppedImgs;

	// an array to hold user's click-points that define the clipping area
	let points = [];

	// variables for undo-ing and redo-ing
	let rpoints = [];
	let prevMove;

	// variables for selecting cropping type
	let cropType = 'rectangle';
	let rectangleBtn, polygonBtn;

	// variables to download all the cropped images
	export let allCrops;
	export let filename;

	onMount(() => {
		ctx = canvas.getContext('2d');
		croppedImgs = document.getElementById('cropped-imgs');

		console.log(typeof cwidth);
	});

	function onFileSelected(e) {
		const reader = new FileReader();
		reader.addEventListener('load', function (evt) {
			var newImg = new Image();
			newImg.onload = function () {
				canvas.classList.remove('hide');
				uploadBtn.classList.add('hide');
				canvas.height = (canvas.width / img.width) * img.height;
				ctx.strokeStyle = '#ED3996';
				ctx.fillStyle = '#ED399650';
				ctx.drawImage(newImg, 0, 0, newImg.width, newImg.height, 0, 0, canvas.width, canvas.height);
			};
			if (evt.target && typeof evt.target.result == 'string') {
				newImg.src = evt.target.result;
			}
			img = newImg;
		});
		filename = e.target.files[0].name.split('.')[0];
		reader.readAsDataURL(e.target.files[0]);
	}

	function setCropType(set) {
		cropType = set;
		if (set == 'rectangle') {
			rectangleBtn.classList.add('selected');
			polygonBtn.classList.remove('selected');
		} else {
			rectangleBtn.classList.remove('selected');
			polygonBtn.classList.add('selected');
		}
		reset();
	}

	function handleMouseDown(e) {
		if (cropType == 'polygon') {
			polygonCrop(e);
		} else if (cropType == 'rectangle') {
			rectangleCrop(e);
		} else {
			console.log('error');
		}
	}

	function polygonCrop(e) {
		// tell the browser that we're handling this event
		e.preventDefault();
		e.stopPropagation();

		// calculate mouseX & mouseY
		let mx = e.pageX - canvas.offsetLeft;
		let my = e.pageY - canvas.offsetTop;

		// push the clicked point to the points[] array
		points = [...points, { x: mx, y: my }];

		// remove points from redo[] array
		rpoints.length = 0;
		rpoints = rpoints;

		// set prevMove
		prevMove = 'polygon';

		// show the user an outline of their current clipping path
		outlineIt();

		// if the user clicked back in the original circle
		// then complete the clip
		if (points.length > 1) {
			var dx = mx - points[0].x;
			var dy = my - points[0].y;
			if (dx * dx + dy * dy < 10 * 10) {
				clipIt();
			}
		}
	}

	function rectangleCrop(e) {
		// tell the browser that we're handling this event
		e.preventDefault();
		e.stopPropagation();

		// calculate mouseX & mouseY
		let mx = e.pageX - canvas.offsetLeft;
		let my = e.pageY - canvas.offsetTop;

		// maximum two clicks
		if (points.length < 2) {
			// push the clicked point to the points[] array and set prevMove
			if (points.length == 0) {
				points = [...points, { x: mx, y: my }];
				prevMove = 'draw1';
			} else {
				points = [
					...points,
					{ x: mx, y: points[0].y },
					{ x: mx, y: my },
					{ x: points[0].x, y: my }
				];
				prevMove = 'draw3';
			}

			// show the user an outline of their current clipping path
			outlineIt();
		}

		// if the user clicked back in the original circle
		// then complete the clip
		if (points.length > 1) {
			var dx = mx - points[0].x;
			var dy = my - points[0].y;
			if (dx * dx + dy * dy < 10 * 10) {
				clipIt();
			}
		}
	}

	// show the current potential clipping path
	function outlineIt() {
		ctx.drawImage(img, 0, 0, img.width, img.height, 0, 0, canvas.width, canvas.height);

		if (points.length > 0) {
			ctx.beginPath();
			ctx.moveTo(points[0].x, points[0].y);
			for (var i = 0; i < points.length; i++) {
				ctx.lineTo(points[i].x, points[i].y);
			}
			ctx.closePath();
			ctx.fill();
			ctx.stroke();

			// draw beginning circle
			ctx.beginPath();
			ctx.arc(points[0].x, points[0].y, 10, 0, Math.PI * 2);
			ctx.closePath();
			ctx.stroke();
		}
	}

	// removes a point from points[] and adds it to rpoints[]
	function undo() {
		if (prevMove == 'draw3') {
			undoo();
			undoo();
			undoo();
			prevMove = 'undo3';
		} else {
			undoo();
			prevMove = 'undo1';
		}
		outlineIt();
	}

	function undoo() {
		var move = points.pop();
		points = points;
		rpoints = [...rpoints, move];
	}

	// removes a point from rpoints[] and adds it to points[]
	function redo() {
		if (prevMove == 'undo3') {
			redoo();
			redoo();
			redoo();
			prevMove = 'draw3';
		} else {
			redoo();
			prevMove = 'draw1';
		}
		outlineIt();
	}

	function redoo() {
		var move = rpoints.pop();
		rpoints = rpoints;
		points = [...points, move];
	}

	function reset() {
		rpoints.length = 0;
		rpoints = rpoints;
		points.length = 0;
		points = rpoints;
		outlineIt();
	}

	// clip the selected path to a new canvas
	function clipIt() {
		// calculate the size of the user's clipping area
		var minX = 10000;
		var minY = 10000;
		var maxX = -10000;
		var maxY = -10000;
		for (var i = 1; i < points.length; i++) {
			var p = points[i];
			if (p.x < minX) {
				minX = p.x;
			}
			if (p.y < minY) {
				minY = p.y;
			}
			if (p.x > maxX) {
				maxX = p.x;
			}
			if (p.y > maxY) {
				maxY = p.y;
			}
		}
		var width = maxX - minX;
		var height = maxY - minY;

		// clip the image into the user's clipping area
		ctx.save();
		ctx.clearRect(0, 0, canvas.width, canvas.height);
		ctx.beginPath();
		ctx.moveTo(points[0].x, points[0].y);
		for (var i = 1; i < points.length; i++) {
			var p = points[i];
			ctx.lineTo(points[i].x, points[i].y);
		}
		ctx.closePath();
		ctx.clip();
		ctx.drawImage(img, 0, 0, img.width, img.height, 0, 0, canvas.width, canvas.height);
		ctx.restore();

		// create a new canvas
		var c = document.createElement('canvas');
		var cx = c.getContext('2d');

		// resize the new canvas to the size of the clipping area
		c.width = width;
		c.height = height;

		// draw the clipped image from the main canvas to the new canvas
		if (cx) {
			cx.drawImage(canvas, minX, minY, width, height, 0, 0, width, height);
		}

		// create a new Image() from the new canvas
		var clippedImage = new Image();
		clippedImage.onload = function () {
			// append the new image to the page
			croppedImgs.appendChild(createDiv(clippedImage));
		};
		clippedImage.src = c.toDataURL();

		// add to allCrops{}
		allCrops = [...allCrops, { id: allCrops.length, points: points.slice() }];

		// clear the previous points
		points.length = 0;

		// redraw the image on the main canvas for further clipping
		ctx.drawImage(img, 0, 0, img.width, img.height, 0, 0, canvas.width, canvas.height);
	}

	// function for adding clipped images to page
	function createDiv(clippedImage) {
		clippedImage.id = 'crop-' + croppedImgs.children.length;

		let deleteBtn = document.createElement('button');
		deleteBtn.id = 'deleteBtn-' + croppedImgs.children.length;
		deleteBtn.classList.add('icon-button');
		deleteBtn.innerHTML =
			'<span class="material-icons-outlined" id="minusIcon-' + croppedImgs.children.length + '">do_not_disturb_on</span>';
		deleteBtn.addEventListener('click', deleteCrop);

		let div = document.createElement('div');
		div.classList.add('cropped-img-div');
		div.id = 'cropDiv-' + croppedImgs.children.length;
		div.appendChild(clippedImage);
		div.appendChild(deleteBtn);

		return div;
	}

	function deleteCrop(e) {
		let idText = e.target.id.split('-');
		let idNum = idText[idText.length - 1];

		allCrops.splice(allCrops.indexOf(allCrops.find((x) => x.id == idNum)), 1);
		allCrops = allCrops;
		let id = document.getElementById('cropDiv-' + idNum);
		if (id) {
			id.remove();
		}
	}

	// function for drawing cross hairs
	function handleMouseMove(e) {
		// tell the browser that we're handling this event
		e.preventDefault();
		e.stopPropagation();

		// calculate mouseX & mouseY
		let mx = e.pageX - canvas.offsetLeft;
		let my = e.pageY - canvas.offsetTop;

		if (img) {
			ctx.clearRect(0, 0, canvas.width, canvas.height);
			ctx.drawImage(img, 0, 0, img.width, img.height, 0, 0, canvas.width, canvas.height);
			outlineIt();

			// draw lines
			ctx.beginPath();
			// draw x-axis
			ctx.moveTo(mx, 0);
			ctx.lineTo(mx, my - 10);
			ctx.moveTo(mx, my + 10);
			ctx.lineTo(mx, canvas.height);
			// draw y-axis
			ctx.moveTo(0, my);
			ctx.lineTo(mx - 10, my);
			ctx.moveTo(mx + 10, my);
			ctx.lineTo(canvas.width, my);
			ctx.closePath();
			ctx.stroke();
		}
	}

	// function for removing the selected image
	function deleteImage() {
		canvas.classList.add('hide');
		uploadBtn.classList.remove('hide');
		input.value = null;
		allCrops = [];
		reset();
		img = undefined;
		filename = undefined;
		while (croppedImgs.firstChild) {
			croppedImgs.removeChild(croppedImgs.firstChild);
		}
	}
</script>

<div class="section cropper-wrapper">
	<div class="canvas">
		<canvas
			bind:this={canvas}
			width={cwidth}
			height={cheight}
			on:mousedown={(e) => handleMouseDown(e)}
			on:mousemove={(e) => handleMouseMove(e)}
			class="hide"
		/>
		<button
			class="upload-button"
			bind:this={uploadBtn}
			on:click={input.click()}
			style={'width:' + cwidth + 'px;height:' + cheight + 'px;'}
		>
			<span class="fa-solid fa-images" />
			<h6>Upload image</h6>
		</button>
		<input
			type="file"
			accept=".jpg, .jpeg, .png"
			class="hide"
			bind:this={input}
			on:change={(e) => onFileSelected(e)}
		/>
	</div>
	<div class="buttons">
		<button
			bind:this={rectangleBtn}
			on:click={() => setCropType('rectangle')}
			disabled={img == undefined}
			class="icon-button selected"
			title="Rectangle Crop"><span class="material-icons-outlined">crop</span></button
		>
		<button
			bind:this={polygonBtn}
			on:click={() => setCropType('polygon')}
			disabled={img == undefined}
			class="icon-button"
			title="Polygon Crop"><span class="material-icons-outlined">hexagon</span></button
		>
		<button on:click={reset} disabled={points.length == 0} class="icon-button" title="Reset"
			><span class="material-icons-outlined">cleaning_services</span></button
		>
		<button on:click={undo} disabled={points.length == 0} class="icon-button" title="Undo"
			><span class="material-icons-outlined">undo</span></button
		>
		<button on:click={redo} disabled={rpoints.length == 0} class="icon-button" title="Redo"
			><span class="material-icons-outlined">redo</span></button
		>
		<button on:click={deleteImage} disabled={img == undefined} class="icon-button" title="Delete"
			><span class="material-icons-outlined">delete</span></button
		>
	</div>
</div>

<style>
	.cropper-wrapper {
		flex: 1 0 500px;

		display: flex;
		justify-content: center;

		gap: 8px;
	}

	.canvas canvas {
		border: 1px solid #d3d3d3;
		cursor: crosshair;
	}

	.canvas .upload-button {
		background: transparent;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		gap: 8px;
		border: 1px solid #d3d3d3;
	}

	.canvas .upload-button span {
		font-size: 64px;
	}

	.buttons {
		display: flex;
		flex-direction: column;
		gap: 8px;
	}

	.selected {
		color: #ed3996;
	}

	button:disabled {
		opacity: 0.5;
	}

	.hide {
		display: none !important;
	}

	:global(.icon-button) {
		background: transparent;
		border: none;
	}

	:global(.icon-button:hover) {
		color: #d51477;
	}

	:global(.icon-button:disabled) {
		color: #404040;
	}

	:global(.icon-button > span) {
		font-size: 32px;
	}

	:global(.cropped-img-div) {
		border: solid 2px #fac7e1;
		padding: 16px;
		border-radius: 8px;
		display: flex;
		justify-content: space-between;
	}
</style>
