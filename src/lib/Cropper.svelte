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
	export let colours;

	// image to be cropped
	let img;

	// div to store cropped images
	let croppedImgs;

	// an array to hold user's click-points that define the clipping area
	let points = [];
	let moves = [];

	// variables for undo-ing and redo-ing
	let rpoints = [];
	let rmoves = [];

	// variables for selecting cropping type
	let cropType = 'rectangle';
	let rectangleBtn, polygonBtn;

	// variables to download all the cropped images
	export let allCrops;
	export let filename;

	onMount(() => {
		ctx = canvas.getContext('2d');
		croppedImgs = document.getElementById('cropped-imgs');
	});

	function onFileSelected(e) {
		const reader = new FileReader();
		reader.addEventListener('load', function (evt) {
			var newImg = new Image();
			newImg.onload = function () {
				canvas.classList.remove('hide');
				uploadBtn.classList.add('hide');
				canvas.height = (canvas.width / img.width) * img.height;
				ctx.drawImage(newImg, 0, 0, newImg.width, newImg.height, 0, 0, canvas.width, canvas.height);

				allCrops = [... allCrops, {cwidth: canvas.width, cheight: canvas.height}];
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
		// check if shiftKey is held
		if (cropType == 'polygon' && e.shiftKey && points.length > 0) {
			// calculate angle of mx and my wrt points[points.length-1] to decide horizontal or vertial line
			let angle = Math.abs(Math.atan2(my - points[points.length-1].y, mx - points[points.length-1].x) * 180 / Math.PI);

			// draw lines
			if (angle > 45 && angle < 135) {
				// draw along y-axis
				points = [...points, { x: points[points.length-1].x, y: my }];
			} else {
				// draw along x-axis
				points = [...points, { x: mx, y: points[points.length-1].y }];
			}
		} else {
			points = [...points, { x: mx, y: my }];
		}

		// remove points from redo[] array
		rpoints.length = 0;
		rpoints = rpoints;

		// add move to moves[]
		moves = [...moves, 'draw1'];

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
			// push the clicked point to the points[] array and set moves
			// add move to moves[]
			if (points.length == 0) {
				points = [...points, { x: mx, y: my }];
				moves = [...moves, 'draw1'];
			} else {
				points = [
					...points,
					{ x: mx, y: points[0].y },
					{ x: mx, y: my },
					{ x: points[0].x, y: my }
				];
				moves = [...moves, 'draw3'];
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
			ctx.strokeStyle = colours.primary;
			ctx.beginPath();
			ctx.moveTo(points[0].x, points[0].y);
			for (var i = 0; i < points.length; i++) {
				ctx.lineTo(points[i].x, points[i].y);
			}
			ctx.closePath();
			ctx.fillStyle = colours.primary+'50';
			ctx.fill();
			ctx.stroke();

			// draw beginning circle
			ctx.beginPath();
			ctx.arc(points[0].x, points[0].y, 10, 0, Math.PI * 2);
			ctx.closePath();
			ctx.stroke();
		}
	}

	// removes a point from moves[] and adds it to rmoves[]
	function undo() {
		var move = moves.pop();
		if (move == 'draw3') {
			undoo();
			undoo();
			undoo();
			rmoves = [...rmoves, move];
		} else {
			undoo();
			rmoves = [...rmoves, move];
		}
		outlineIt();
	}

	// removes a point from points[] and adds it to rpoints[]
	function undoo() {
		var point = points.pop();
		points = points;
		rpoints = [...rpoints, point];
	}

	// removes a point from rmoves[] and adds it to moves[]
	function redo() {
		var move = rmoves.pop();
		if (move == 'draw3') {
			redoo();
			redoo();
			redoo();
			moves = [...moves, move];
		} else {
			redoo();
			moves = [...moves, move];
		}
		outlineIt();
	}

	// removes a point from rpoints[] and adds it to points[]
	function redoo() {
		var point = rpoints.pop();
		rpoints = rpoints;
		points = [...points, point];
	}

	function reset() {
		points.length = 0;
		points = rpoints;
		rpoints.length = 0;
		rpoints = rpoints;
		moves.length = 0;
		moves = moves;
		rmoves.length = 0;
		rmoves = rmoves;
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

		// get id of last child to use for new child
		let idNum = 0;
		if (croppedImgs.children.length > 0) {
			let idText = croppedImgs.children[croppedImgs.children.length-1].id.split('-');
			idNum = parseInt(idText[idText.length - 1]) + 1;
		}

		// create a new Image() from the new canvas
		var clippedImage = new Image();
		clippedImage.onload = function () {
			// append the new image to the page
			croppedImgs.appendChild(createDiv(clippedImage, idNum));
		};
		clippedImage.src = c.toDataURL();

		// add to allCrops{}
		allCrops = [...allCrops, { id: idNum, base64: c.toDataURL(), points: points.slice() }];

		// clear the previous points
		points.length = 0;

		// redraw the image on the main canvas for further clipping
		ctx.drawImage(img, 0, 0, img.width, img.height, 0, 0, canvas.width, canvas.height);
	}

	// function for adding clipped images to page
	function createDiv(clippedImage, idNum) {
		clippedImage.id = 'crop-' + idNum;

		let deleteBtn = document.createElement('button');
		deleteBtn.id = 'deleteBtn-' + idNum;
		deleteBtn.classList.add('icon-button');
		deleteBtn.innerHTML =
			'<span class="material-icons-outlined" id="minusIcon-' + idNum + '">do_not_disturb_on</span>';
		deleteBtn.addEventListener('click', deleteCrop);

		let div = document.createElement('div');
		div.classList.add('cropped-img-div');
		div.id = 'cropDiv-' + idNum;
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
		}

		// draw vertical and horizontal 
		if (cropType == 'polygon' && e.shiftKey && points.length > 0) {
			// calculate angle of mx and my wrt points[points.length-1] to decide horizontal or vertial line
			let angle = Math.abs(Math.atan2(my - points[points.length-1].y, mx - points[points.length-1].x) * 180 / Math.PI);

			// draw lines
			if (angle > 45 && angle < 135) {
				drawAxis(points[points.length-1].x, my);

				// draw potential line along y-axis
				ctx.strokeStyle = colours.primary;
				ctx.beginPath();
				ctx.moveTo(points[points.length-1].x, points[points.length-1].y)
				ctx.lineTo(points[points.length-1].x, my);
				ctx.closePath();
				ctx.stroke();
			} else {
				drawAxis(mx, points[points.length-1].y);

				// draw potential line along x-axis
				ctx.strokeStyle = colours.primary;
				ctx.beginPath();
				ctx.moveTo(points[points.length-1].x, points[points.length-1].y)
				ctx.lineTo(mx, points[points.length-1].y);
				ctx.closePath();
				ctx.stroke();
			}
		} else if (cropType == 'polygon' && points.length > 0) {
			drawAxis(mx, my);

			// draw potential line
			ctx.strokeStyle = colours.primary;
			ctx.beginPath();
			ctx.moveTo(points[points.length-1].x, points[points.length-1].y)
			ctx.lineTo(mx, my);
			ctx.closePath();
			ctx.stroke();
		} else if (cropType == 'rectangle' && points.length == 1) { 
			drawAxis(mx, my);

			// draw potential line
			ctx.strokeStyle = colours.primary;
			ctx.beginPath();
			ctx.moveTo(points[points.length-1].x, points[points.length-1].y)
			ctx.lineTo(points[points.length-1].x, my);
			ctx.lineTo(mx, my);
			ctx.lineTo(mx, points[points.length-1].y);
			ctx.closePath();
			ctx.stroke();
		} else {
			drawAxis(mx, my);
		}

		drawTooltip(mx, my);
	}

	// fucntion to draw axis lines
	function drawAxis(mx, my) {
		ctx.strokeStyle = colours.secondary;
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

	// function to draw tooltip
	function drawTooltip(mx, my) {
		ctx.fillStyle = colours.tooltip;
		// check if tooltip will be blocked
		let tooltip_placement_x = 10;
		let tooltip_placement_y = 10;
		if (mx+10+76 > canvas.width) tooltip_placement_x = -86; //-10-76
		if (my+10+20 > canvas.height) tooltip_placement_y = -30; //-10-20
		ctx.fillRect(mx+tooltip_placement_x, my+tooltip_placement_y, 76, 20);
		ctx.fillStyle = colours.tooltip_text;
		ctx.font = 'normal 16px';
		ctx.fillText('x: '+mx+', y: '+my, mx+tooltip_placement_x+8, my+tooltip_placement_y+14);
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

<div class="section cropper-wrapper"
	style="--primary-color: {colours.primary};
	--secondary-color: {colours.secondary};
	--cwidth: {cwidth}px;"
>
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
			<span class="material-icons-outlined">upload_file</span>
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
		flex: 1 0 var(--cwidth);

		display: flex;
		justify-content: center;

		gap: 8px;
	}

	.canvas canvas {
		border: 1px solid #d3d3d3;
		cursor: crosshair;
	}

	.canvas .upload-button {
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
		color: var(--primary-color) !important;
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
		color: #404040;
	}

	:global(.icon-button:hover) {
		color: var(--secondary-color);
	}

	:global(.icon-button:disabled) {
		color: #404040;
	}

	:global(.icon-button > span) {
		font-size: 32px;
	}

	:global(.cropped-img-div) {
		border: solid 2px #404040;
		padding: 16px;
		border-radius: 8px;
		display: flex;
		justify-content: space-between;
	}
</style>
