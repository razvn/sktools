<script lang="ts">
	let value: string = '';
	let checkMessage: string = '';
	let index = 0;
	let bloc: string[] = [];
	let errors = new Map<string, number>();
	let filename = "file"
	const extension = "srt"

	function checkValid(input: string, scroll: boolean = false) {
		if (input) {
			resetValues();
			let currentLine = 0;
            let lastStart = 0
			value.split(/\r?\n/).forEach((line) => {
				if (line.trim()) {
					bloc.push(line);
				} else {
					let error = analyse(index, bloc);
					if (error.length) {
						error.forEach((e) => errors.set(e, lastStart + 1));
					}
					bloc = [];
					index++;
                    lastStart = currentLine
				}
                currentLine++
			});
			checkMessage = `${new Date().toLocaleTimeString('fr-FR')} - ${
				errors.size ? 'Errors' : 'No error'
			}`;
		} else {
			checkMessage = '';
		}

		errors = errors;
		if (scroll) window.scrollTo(0, 0);
	};

	function analyse(index: number, bloc: string[]): string[] {
		// console.log('Analyse bloc', bloc.length, bloc);
		const err: string[] = [];
		if (bloc.length < 3) {
			err.push(`[${index + 1}]: Invalid bloc size: '${bloc.length}'`);
			return err;
		}
		let idx = bloc[0];
		if (idx != `${index + 1}`) {
			err.push(`[${idx}]: Index should be '${index + 1}'`);
		}
		let time = bloc[1];
		if (!time || !time.match(/\d{2}:\d{2}:\d{2},\d{3} --> \d{2}:\d{2}:\d{2},\d{3}/)) {
			err.push(`[${idx}]: Timestamp '${time}' is not the right format`);
		}
		return err;
	};

	function resetValues() {
		index = 0;
		bloc = [];
		errors.clear();
	};

	const scrollValue: number = 18;

	let scrollTxt = ''

	function scroll(value: number){
		const textarea = document.getElementById('text_id');
		if (textarea) textarea.scrollTop = value * scrollValue;
		updateScrollTxt()
	};

	function scrollTop() {
		const textarea = document.getElementById('text_id');
		if (textarea) textarea.scrollTop = 0
		updateScrollTxt()
	}

	function scrollBottom() {
		const textarea = document.getElementById('text_id');
		if (textarea) textarea.scrollTop =  textarea.scrollHeight
		updateScrollTxt()
	}
	
	function scrollPosition(): string {
		const textarea = document.getElementById('text_id');
		if (textarea) {
			if (textarea.scrollTop == 0) return 'top'
			else return `${textarea.scrollTop} - ${textarea.scrollHeight}`
		} 
		return ''
	}


	let lineNo = ''
	function updateLineNo() {
		const textarea = document.getElementById('text_id') as HTMLTextAreaElement;
		if (textarea && textarea.value && textarea.selectionStart) {
			lineNo = `${textarea.value.substring(0, textarea.selectionStart).split(/\r?\n|\r/).length}`
		}
	}

	function updateScrollTxt() { 
		scrollTxt = scrollPosition() 
	}	

	function save(content: string, filename: string, extension: string) {
		const blob = new Blob([content], {
   				type: "text/plain;charset=utf-8",
		});
		const link = document.createElement("a");
		link.href = URL.createObjectURL(blob);
         link.download = `${filename}.${extension}`;
         link.click();
         URL.revokeObjectURL(link.href);
	}

</script>

<h1>SRT Validation tool</h1>
<div style="padding: 10px;">
	
</div>
<div class="flex-container">
	<div class="flex-child" >
		<div style="padding: 10px;" class="flex-container" >
			<div class="flex-child" style="width: 100%;">
				<button on:click={() => checkValid(value, true)}>Validate</button>
			</div>
			<div class="flex-child" style="flex: 0 auto">
				<span>{scrollTxt}</span>
				<a href="#top" on:click|preventDefault={() => scrollTop()}>⬆️</a>
				<a href="#top" on:click|preventDefault={() => scrollBottom()}>⬇️</a>
			</div>
		</div>
		<textarea id="text_id" bind:value 
		on:change={updateScrollTxt} 
		on:scroll={updateScrollTxt} 
		on:click={updateLineNo} 
		on:keypress={updateLineNo}
		on:dblclick={updateLineNo}
		on:focus={updateLineNo}
		on:select={updateLineNo}
		on:keyup={updateLineNo}
		on:focusout={updateLineNo} 
		on:abort={updateLineNo}/>
		<div class="flex-container">
			{#if lineNo}
			<span>line: {lineNo}</span>
			{/if}
		</div>
		<div style="padding: 10px;" class="flex-container" >
			<div class="flex-child" style="flex: 0">
				<button on:click={() => checkValid(value, true)}>Validate</button>
			</div>
			<div class="flex-child" style="flex: 2; white-space: nowrap; padding-right: 2em;">
				<input type="text" bind:value={filename} />.{extension}
			</div>
			<div class="flex-child" style="flex: 0;">
				<button on:click={() => save(value, filename, extension)}>Save file</button>
			</div>
		</div>
	</div>
	<div class="flex-child {errors.size ? 'ko' : 'ok'}">
		<h2>{checkMessage}</h2>
		{#if errors.size}
			<ul>
				{#each [...errors] as [error, line]}
					<li>{error} (<button class="link" on:click={() => scroll(line)}>Go to line {line}</button>)</li>
				{/each}
			</ul>
		{/if}
	</div>
</div>


<style>
	textarea {
		width: 100%;
		height: 600px;
		font-size: 16px;
		font-family: 'Courier New', Courier, monospace;
	}

	input {
		font-size: 16px;
		text-align: right;
		width: 100%;
		font-family: 'Courier New', Courier, monospace;
	}

	.flex-container {
		display: flex;
		font-family: 'Courier New', Courier, monospace;
		align-items: center;
	}

	.flex-child {
		flex: 1;
		margin-right: 20px;
	}

	.ok {
		color: green;
	}

	.ko {
		color: red;
	}

	button {
		appearance: none;
		background-color: #2ea44f;
		border: 1px solid rgba(27, 31, 35, 0.15);
		border-radius: 6px;
		box-shadow: rgba(27, 31, 35, 0.1) 0 1px 0;
		box-sizing: border-box;
		color: #fff;
		cursor: pointer;
		display: inline-block;
		font-family: -apple-system, system-ui, 'Segoe UI', Helvetica, Arial, sans-serif,
			'Apple Color Emoji', 'Segoe UI Emoji';
		font-size: 14px;
		font-weight: 600;
		line-height: 20px;
		padding: 6px 16px;
		position: relative;
		text-align: center;
		text-decoration: none;
		user-select: none;
		-webkit-user-select: none;
		touch-action: manipulation;
		vertical-align: middle;
		white-space: nowrap;
	}

	button:focus:not(:focus-visible):not(.focus-visible) {
		box-shadow: none;
		outline: none;
	}

	button:hover {
		background-color: #2c974b;
	}

	button:focus {
		box-shadow: rgba(46, 164, 79, 0.4) 0 0 0 3px;
		outline: none;
	}

	button:disabled {
		background-color: #94d3a2;
		border-color: rgba(27, 31, 35, 0.1);
		color: rgba(255, 255, 255, 0.8);
		cursor: default;
	}

	button:active {
		background-color: #298e46;
		box-shadow: rgba(20, 70, 32, 0.2) 0 1px 0 inset;
	}

	.link {
		background: none !important;
		border: none;
		padding: 0 !important;
		/*input has OS specific font-family*/
		color: #069;
		text-decoration: underline;
		cursor: pointer;
	}
</style>
