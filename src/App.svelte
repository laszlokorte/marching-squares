<script>
	const sideLength = 1000
	let divisions = 40
	let step = divisions*divisions-1
	let offset = 0
	let dx = 2
	let dy = 100
	let radius = 100
	let dxB = 200
	let dyB = -103
	let radiusB = 100
	let showIso = false

	let showOutlineA = true
	let showOutlineB = true

	let sdf = false

	function setDivisions(val) {
		const shrink = val < divisions
		const oldRow = (step % divisions) / divisions
		const oldCol = (Math.floor(step / divisions)) / divisions

		const newCol = shrink ? Math.floor(oldCol * val) : Math.ceil(oldCol * val)
		const newRow = shrink ? Math.floor(oldRow * val)  : Math.ceil(oldRow * val)
		
		divisions = val

		step = newCol * val + newRow
	}
	
	function isoShapes(isoValue, hitCenter, north, east, south, west) {
		switch(isoValue) {
			case 0:
				return []
			case 1:
				return [[west.x, west.y, south.x, south.y, west.x, south.y]]
			case 2:
				return [[south.x, south.y, east.x, east.y, east.x, south.y]]
			case 3:
				return [[east.x, east.y, west.x, west.y, west.x, south.y, east.x, south.y]]
			case 12:
				return [[east.x, east.y, west.x, west.y, west.x, north.y, east.x, north.y]]
			case 4:
				return [[north.x, north.y, east.x, east.y, east.x, north.y]]
			case 11:
				return [[north.x, north.y, east.x, east.y, east.x, south.y, west.x, south.y, west.x, north.y]]
			case 5:
				if(hitCenter) {
					return [[north.x, north.y, west.x, west.y, west.x, south.y, south.x, south.y, east.x, east.y, east.x, north.y]]
				} else {
					return [
						[east.x,east.y, north.x, north.y, east.x, north.y],
						[west.x,west.y, south.x, south.y, west.x, south.y],
					]
				}
			case 6:
				return [[north.x, north.y, south.x, south.y, east.x, south.y, east.x, north.y]]
			case 9:
				return [[north.x, north.y, south.x, south.y, west.x, south.y, west.x, north.y]]
			case 7:
				return [[north.x, north.y, west.x, west.y, west.x, south.y, east.x, south.y, east.x, north.y]]
			case 8:
				return [[north.x, north.y, west.x, west.y, west.x, north.y]]
			case 10:
				if(hitCenter) {
					return [[north.x, north.y, east.x, east.y, east.x, south.y,  south.x, south.y, west.x, west.y, west.x, north.y]]
				} else {
					return [
						[west.x, north.y, north.x, north.y, west.x, west.y],
						[east.x, east.y, east.x, south.y, south.x, south.y]
					]
				}
			case 13:
				return [[east.x, east.y, south.x, south.y, west.x, south.y, west.x, north.y, east.x, north.y, east.x, south.y]]
			case 14:
				return [[west.x, west.y, south.x, south.y, east.x, south.y, east.x, north.y, west.x, north.y]]
			case 15:
				return [[west.x, north.y, east.x, north.y, east.x, south.y, west.x, south.y]]
			default:
				return [];
		}
	}
	
	function isoOutlines(isoValue, hitCenter, north, east, south, west) {
		switch(isoValue) {
			case 0:
			case 15:
				return []
			case 1:
				return [[west.x, west.y, south.x, south.y]]
			case 2:
				return [[south.x, south.y, east.x, east.y]]
			case 3:
			case 12:
				return [[east.x, east.y, west.x, west.y]]
			case 4:
			case 11:
				return [[north.x, north.y, east.x, east.y]]
			case 5:
				if(hitCenter) {
					return [[north.x, north.y, west.x, west.y], [south.x, south.y, east.x, east.y]]
				} else {
					return [[north.x, north.y, east.x, east.y], [south.x, south.y, west.x, west.y]]
				}
			case 6:
			case 9:
				return [[north.x, north.y, south.x, south.y]]
			case 7:
			case 8:
				return [[north.x, north.y, west.x, west.y]]
			case 10:
				if(hitCenter) {
					return [[north.x, north.y, east.x, east.y], [south.x, south.y, west.x, west.y]]
				} else {
					return [[north.x, north.y, west.x, west.y], [south.x, south.y, east.x, east.y]]
				}
			case 13:
				return [[east.x, east.y, south.x, south.y]]
			case 14:
				return [[west.x, west.y, south.x, south.y]]
			default:
				return [];
		}
	}
	
	function lerp(t, a,b) {
		return a + (b-a) * t
	}
	
	function sampleSquare(func, {x,y,width, height}) {
		const v1 = func(x,y)
		const v2 = func(x+width,y)
		const v3 = func(x+width,y+height)
		const v4 = func(x,y+height)
		
		const hit1 = v1 >= 1
		const hit2 = v2 >= 1
		const hit3 = v3 >= 1
		const hit4 = v4 >= 1

		
		const isoValue = (hit1?1:0) * 8 + (hit2?1:0) * 4 + (hit3?1:0) * 2 + (hit4?1:0) * 1
		

		const hitCenter = (isoValue === 5 || isoValue === 10) && func(x+width/2,y+height/2) >= 1

		const northLerp = (1-v1)/(v2-v1) || 0
		const southLerp = (1-v4)/(v3-v4) || 0
		const eastLerp = (1-v2)/(v3-v2) || 0
		const westLerp = (1-v1)/(v4-v1) || 0
		
		const north = {y: y, x: lerp(northLerp, x, x+width)}
		const south = {y: y+height, x: lerp(southLerp, x, x+width)}
		const east = {y: lerp(eastLerp, y, y+height), x: x+width}
		const west = {y: lerp(westLerp, y, y+height), x: x}
		
		
		return {
			shapes: isoShapes(isoValue, hitCenter, north, east, south, west),
			outlines: isoOutlines(isoValue, hitCenter, north, east, south, west),
			hits: [hit1, hit2, hit3, hit4],
			isoValue: isoValue,
			lerped: {north, south, east, west},
		}
	}

	function sampleSquareSDF(func, {x,y,width, height}) {
		const v1 = func(x,y)
		const v2 = func(x+width,y)
		const v3 = func(x+width,y+height)
		const v4 = func(x,y+height)
		
		const hit1 = v1 <= 0
		const hit2 = v2 <= 0
		const hit3 = v3 <= 0
		const hit4 = v4 <= 0

		
		const isoValue = (hit1?1:0) * 8 + (hit2?1:0) * 4 + (hit3?1:0) * 2 + (hit4?1:0) * 1
		

		const hitCenter = (isoValue === 5 || isoValue === 10) && func(x+width/2,y+height/2) <= 0

		const northLerp = (v1)/(v1-v2) || 0
		const southLerp = (v4)/(v4-v3) || 0
		const eastLerp = (v2)/(v2-v3) || 0
		const westLerp = (v1)/(v1-v4) || 0
		
		const north = {y: y, x: lerp(northLerp, x, x+width)}
		const south = {y: y+height, x: lerp(southLerp, x, x+width)}
		const east = {y: lerp(eastLerp, y, y+height), x: x+width}
		const west = {y: lerp(westLerp, y, y+height), x: x}
		
		
		return {
			shapes: isoShapes(isoValue, hitCenter, north, east, south, west),
			outlines: isoOutlines(isoValue, hitCenter, north, east, south, west),
			hits: [hit1, hit2, hit3, hit4],
			isoValue: isoValue,
			lerped: {north, south, east, west},
		}
	}
	
	const translate = (dx,dy, shape) => {
		return (x,y) => shape(x-dx, y-dy)
	}
	
	const circleClassic = (r) => {
		const r2 = r*r
		return (x,y) => r2/((x)*(x)+(y)*(y))
	}
	
	const circleSDF = (r) => {
		return (x,y) => Math.sqrt((x)*(x)+(y)*(y)) - r
	}
	
	const add = (s1, s2) => {
		return (x,y) => s1(x,y) + s2(x,y)
	}
	
	const sub = (s1, s2) => {
		return (x,y) => s1(x,y) - s2(x,y)
	}
	
	const min = (s1, s2) => {
		return (x,y) => Math.min(s1(x,y), s2(x,y))
	}
	const max = (s1, s2) => {
		return (x,y) => Math.max(s1(x,y), s2(x,y))
	}
	
	const addOffset = (o, shape) => {
		return (x,y) => shape(x,y) + o
	}

	$: combineShapes = sdf ? min : add

	$: circle = sdf ? circleSDF : circleClassic
	
	$: func = addOffset((sdf?offset:-offset/100), combineShapes(
		translate(dx, dy, circle(radius)),
		translate(dxB, dyB, circle(radiusB))
	))
	
	$: squareLength = sideLength/divisions
	
	$: squares = Array(divisions * divisions).fill(null).map((_,i) => ({
		x: Math.floor(i/divisions)*squareLength - sideLength/2, 
		y: i%divisions*squareLength - sideLength/2,
		width: squareLength,
		height: squareLength,
		show:  i<=step,
		active:  i==step,
	})).map((square) => ({
		...square,
		result: sdf ? sampleSquareSDF(func, square) : sampleSquare(func, square),
	}))
	
	function jumpToStep(evt) {
		if(evt.target.hasAttribute('data-step')) {
			step = parseInt(evt.target.getAttribute('data-step'), 10)
		}
	}
	
</script>

<style>

	fieldset {
		margin:  0;
	}

	svg {
		display: block;
		border: 1px solid gray;
		flex-grow: 1;
		flex-shrink: 1;
		flex-basis: 30em;
	}
	
	input {
		margin: 0;
		padding: 0.2em 0;
		flex-grow: 1;
	}
	
	dl {
		margin: 0;
		display: grid;
		grid-template-columns: max-content auto 4em;
		align-items: center;
		gap: 0.5em 1em;
		padding: 0.5em;
	}
	
	dt {
		grid-column: 1;
	}
	
	dd {
		margin: 0;
		display: flex;
	}
	
	h1 {
		margin: 0;
	}
	
	.intro {

	}

	.container {
		max-width: 60em;
		margin: auto;
		display: flex;
		flex-wrap: wrap;
		gap: 1em;
		padding-bottom: 3em;
	}
	.controls {
		flex-grow: 1;
		flex-basis: 10em;
		gap: 1em;
		display: flex;
		flex-direction: column;
	}

	.check-row {
		display: flex;
		gap: 1em;
		grid-column: span 2;
	}

	legend {
		background-color: #333;
		color: #fff;
		padding: 0.3em;
	}
</style>


<div class="container">

	<div class="intro">
		<h1>
			Marching Squares
		</h1>

		<p>
			Above you can see how an implicit function describing two circles is converted into a polygon moving a square across the shape and sampling the 4 vertices in each step. Each corner of the square can be either inside or outside of the circles (16 possible cases, labeled by the iso numbers). Each possible case results in a specific edge being created or not.
		</p>
		<p>
			The Divisions slider controls the size of the square and by this the number of steps and the resulting resolution.
		</p>
		<p>
		The Step slider highlights the square position in a specified step. The squares corners are marked as being inside (green) or outside (red) the target shape. The orange dots show the linear interpolated intersection points of the square&apos;s edges and the target shape.
		</p>
		<p>
			The target shape consists of two circles. They can be moved and resized via the Shape sliders.
		</p>
	</div>

	<div class="controls">
		<fieldset>
			<legend>
				Marching Square
			</legend>
			
			<dl>

				<dt><label for="type">Function Type</label></dt>
			<dd class="check-row">
				<label><input type="radio" bind:group={sdf} value={false}> Inverse Distance</label>
				<label><input type="radio" bind:group={sdf} value={true}> SDF</label>
		</dd>


			<dt><label for="divisions">Divisions</label></dt>
			<dd><input id="divisions" type="range" min={4} max={50} value={divisions} step="1" on:input={(evt) => setDivisions(evt.currentTarget.valueAsNumber)} />
		</dd>
			<dd><output>{divisions}</output></dd>
			
			<dt><label for="step">Step</label></dt>
			<dd><input id="step" type="range" min={0} max={divisions*divisions - 1} bind:value={step} step="1" />
		</dd>
			<dd><output>{step}</output></dd>
				
			<dt><label for="step">Offset</label></dt>
			<dd><input id="step" type="range" min={-50} step={1} max={50} bind:value={offset} />
		</dd>
			<dd><output>{offset}</output></dd>
				
			<dt></dt>
			<dd class="check-row">
				<label><input type="checkbox" bind:checked={showIso} /> Show Iso Numbers</label>
			</dd>
			</dl>
		</fieldset>

		<fieldset>
			<legend>
				Shape (Circle A)
			</legend>
			
		<dl>	
			<dt><label for="dx">Center X</label></dt>
			<dd><input id="dx" type="range" min={-400} max={400} bind:value={dx} step="1" />
		</dd>
			<dd><output>{dx}</output></dd>
			
			<dt><label for="dx">Center Y</label></dt>
			<dd><input id="dy" type="range" min={-400} max={400} bind:value={dy} step="1" />
		</dd>
			<dd><output>{dy}</output></dd>
			
			<dt><label for="dx">Radius</label></dt>
			<dd><input id="dy" type="range" min={50} max={1000} bind:value={radius} step="1" />
		</dd>
			<dd><output>{radius}</output></dd>
			<dt></dt>
			<dd class="check-row">
				<label><input type="checkbox" bind:checked={showOutlineA} /> Show outline</label>
			</dd>
		</dl>
		</fieldset>


		<fieldset>
			<legend>
				Shape (Circle B)
			</legend>
			
		<dl>	
			<dt><label for="dx">Center X</label></dt>
			<dd><input id="dx" type="range" min={-400} max={400} bind:value={dxB} step="1" />
		</dd>
			<dd><output>{dxB}</output></dd>
			
			<dt><label for="dx">Center Y</label></dt>
			<dd><input id="dy" type="range" min={-400} max={400} bind:value={dyB} step="1" />
		</dd>
			<dd><output>{dyB}</output></dd>
			
			<dt><label for="dx">Radius</label></dt>
			<dd><input id="dy" type="range" min={50} max={1000} bind:value={radiusB} step="1" />
		</dd>
			<dd><output>{radiusB}</output></dd>
			<dt></dt>
			<dd class="check-row">
				<label><input type="checkbox" bind:checked={showOutlineB} /> Show outline</label>
			</dd>
		</dl>
		</fieldset>

	</div>

<svg viewBox="{-(sideLength + 40)/2} {-(sideLength + 40)/2} {sideLength + 40} {sideLength + 40}" on:click={jumpToStep}>
	

	{#if showOutlineA}
	<circle cx={dx} cy={dy} r={radius} fill="none" stroke="gray" stroke-dasharray="20 20" stroke-width="3" />
	{/if}

	{#if showOutlineB}
	<circle cx={dxB} cy={dyB} r={radiusB} fill="none" stroke="gray" stroke-dasharray="20 20" stroke-width="3" />
	{/if}

	{#each squares as {x,y,width,height,active, show, result}, i}	
		{#if show}
		{#each result.shapes as shape}
		<polygon points={shape.join(' ')} opacity="0.5" fill="#00ddaa" />		{/each}
		{#each result.outlines as outline}
		<polyline points={outline.join(' ')} fill="none" stroke="black" />
		{/each}
		{#if showIso}
			<text x={x+width/2} y={y+height/2} text-anchor="middle" dominant-baseline="middle">{result.isoValue}</text>
		{/if}
		{/if}

		{#if active}
		<rect x={x} y={y} width={width} height={height} stroke="black" fill="none" stroke-dasharray="5 5" />
		<circle cx={x} cy={y} r="4" fill={result.hits[0] ? 'green' : 'red'} />
		<circle cx={x+width} cy={y} r="4" fill={result.hits[1] ? 'green' : 'red'} />
		<circle cx={x+width} cy={y+height} r="4" fill={result.hits[2] ? 'green' : 'red'} />
		<circle cx={x} cy={y+height} r="4" fill={result.hits[3] ? 'green' : 'red'} />
			
			{#if result.hits[0] != result.hits[1]}
				<circle cx={result.lerped.north.x} cy={result.lerped.north.y} r="3" fill="orange" />
			{/if}
		    {#if result.hits[2] != result.hits[3]}
				<circle cx={result.lerped.south.x} cy={result.lerped.south.y} r="3" fill="orange" />
			{/if}
		    {#if result.hits[1] != result.hits[2]}
				<circle cx={result.lerped.east.x} cy={result.lerped.east.y} r="3" fill="orange" />
			{/if}
		    {#if result.hits[3] != result.hits[0]}
				<circle cx={result.lerped.west.x} cy={result.lerped.west.y} r="3" fill="orange" />
			{/if}
		{/if}


		<rect x={x} y={y} width={width} height={height} stroke="none" fill="none" pointer-events="all" data-step={i} />
	{/each}
</svg>

</div>


