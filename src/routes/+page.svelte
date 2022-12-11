<script lang='ts'>
	import { onMount } from 'svelte';
    import { fly } from 'svelte/transition'

	import FloatingWord from '$lib/components/floatingWord.svelte';

    import HSK1 from '$lib/data/hsk/hsk1.json'
    import HSK2 from '$lib/data/hsk/hsk2.json'
    import HSK3 from '$lib/data/hsk/hsk3.json'
    import HSK4 from '$lib/data/hsk/hsk4.json'
    import HSK5 from '$lib/data/hsk/hsk5.json'
    import HSK6 from '$lib/data/hsk/hsk6.json'

    type HSKWord = {
        "simplified": string,
        "traditional": string,
        "pinyin": string,
        "definitions": string[];
    }

    // Words are moved from wordList to displayedWords
    let wordList : HSKWord[] = HSK1;
    let displayedWords : HSKWord[] = []
    let snipedWords : HSKWord[] = []
    let missedWords : HSKWord[] = []

    let levels = ["1","2","3","4","5","6"]
	let selectedLevel = "1";
    // Set HSK level
	function setLevel(level: string) {
		selectedLevel = level
        switch (level) {
            case "1":
                wordList = HSK1
                break;
            case "2":
                wordList = HSK2
                break;
            case "3":
                wordList = HSK3
                break;
            case "4":
                wordList = HSK4
                break;
            case "5":
                wordList = HSK5
                break;
            case "6":
                wordList = HSK6
                break;
        }
        displayedWords = []
        snipedWords = []
        missedWords = []
	}

    // Interval for selecting random words
    let wordInterval : NodeJS.Timer 

    function selectRandomWord () {
          // Select a random word from the word list
		let index = Math.floor(Math.random() * wordList.length);
		let selectedWord = wordList[index];
        
        // Create a new array that doesn't include the selected word
        let remainingWords = wordList.filter((word, i) => i !== index);
    
        // Add the selected word to displayedWords and update the remaining words
        displayedWords = [...displayedWords, selectedWord]
        wordList = remainingWords

        // If there are no more words in the list, clear the interval
		if(!wordList.length) {
			clearInterval(wordInterval)
		}
	}

    // When the component is mounted, select a random word and start the interval
    onMount(() => {
        selectRandomWord()
        wordInterval = setInterval(() => {
            selectRandomWord()}, 
        1 * 5000);
	})
	
    // Define reactive variables to track the remaining words, successful words, and missed words
    $: remainingCount = wordList.length
	$: successCount = snipedWords.length
	$: missedCount = missedWords.length

    let inputEl: HTMLElement;
    // Current user input passed to each floating word as prop
	let currentInput : string;
    // Variables duplicate and show input for smooth animation on snipe
    let inputDuplicate : string = "";
    let showMatch : boolean = true;
	
    // Handle sniped and missed words
	function handleMatch(event: CustomEvent) {
        // Clear input box with smooth animation
        inputDuplicate = event.detail.word.pinyin
        showMatch = true
        currentInput = ""
        setTimeout(() => {showMatch = false}, 10)
        setTimeout(() => {inputDuplicate = ""}, 10)
        // Keep track of snipedwords
		snipedWords.push(event.detail.word)
		snipedWords = snipedWords
        inputEl.focus()
	}

	function handleMiss(event: CustomEvent) {
		missedWords.push(event.detail.word)
		missedWords = missedWords
	}
</script>

<div class="stats">
    <div class="rounded-rectangle" id="sniped">
        <div class="descriptor" id="sniped-text">Sniped:</div>
        <div class="score" id="sniped-score">
            {successCount}
        </div>
    </div>

    <div class="rounded-rectangle" id="rem">
        <div class="descriptor" id="rem-text">Remaining:</div>
        <div class="score" id="rem-score">
            {remainingCount}
        </div>
    </div>

    <div class="rounded-rectangle" id="missed">
        <div class="descriptor" id="missed-text">Missed:</div>
        <div class="score" id="missed-score">
            {missedCount}
        </div>
    </div>
</div>

<div class="hsk-levels">
	<div class="hsk-title">HSK<br>level</div>
	{#each levels as level}
		<!-- svelte-ignore a11y-click-events-have-key-events -->
		<div 
            class="circle" id="hsk-{level}" 
            class:selected="{selectedLevel == level}" 
            on:click="{() => {setLevel(level)}}">
            {level}
		</div>
	{/each}
</div>

<div class="user-input">
    <!-- svelte-ignore a11y-autofocus -->
    <input 
        class="input-box" 
        bind:value={currentInput} 
        bind:this={inputEl}
        placeholder="pinyin"
        autofocus
    />
    {#if showMatch}
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <div 
            class="input-box" 
            id="fake-input"
            out:fly="{{duration: 1000, y: -100}}"
            on:click="{() => {inputEl.focus()}}"
        >
            {inputDuplicate}
        </div>
    {/if}
</div>

<div id="container">
    {#each displayedWords as word (word.simplified)}
        <FloatingWord 
            on:match={handleMatch}
            on:miss={handleMiss}
            currentInput={currentInput} 
            word={word}
        />
    {/each}
</div>

<style>
    .stats {
        position: absolute;

        display: flex;
        flex-direction: row;
        justify-content: space-around;

        margin-top: 8px;
        width: 100%;
    }

    .hsk-levels {
        position: absolute;
        z-index: 5;
        top: 50%;
        left: 8px;
        transform: translate(0, -50%);

		width: 42px;

        font-size: 18px;
		font-family: 'Futura';
		text-align: center;
	}

	.circle {
        border-radius: 20px;
		margin-top: 6px;
		width: 40px;
		height: 40px;

        line-height: 40px;

		background-color: lightgray;
	}

	.selected {
		background-color: red;
	}

    .user-input {
        position: absolute;
        z-index: 10;
        left: 50%;
        top: 50%;

        transform: translate(-50%, -50%);
    }

    input {
        border: 2px solid rgba(204, 0, 0, 0.8);
        border-radius: 6px;

        background: rgba(204, 0, 0, 0.2);
    }

    input:focus {
        outline: red;
    }

    input:focus::placeholder {
        color: transparent;
    }

    .input-box {
        width: 240px;
        height: 60px;

        font-size: 34px;
        line-height: 60px;
        font-family: serif;
        text-align: center;
    }

    #fake-input {
        position: absolute;
        bottom: 2px;
    }

	#container {
        position: relative;

		width: 100%;
        height: calc(100% - 50px);
	}

    .rounded-rectangle {
		display: flex;
		flex-direction: row;

        border-radius: 20px;
        width: fit-content;
		height: 40px;
	}

	.descriptor {
		margin-left: 10px;		
		margin-right: 4px;
		width: auto;

        font-size: 18px;
		line-height: 40px;
		font-family: 'Futura';
	}

	.score {
        margin-right: 4px;
		margin-top: 4px;
        border-radius: 16px;
        width: 50px;
        height: 32px;
        
		font-size: 20px;
		line-height: 32px;
        font-family: 'Futura';
		font-weight: 400;
        text-align: center;
        color: black;
	}
	
	#sniped {
		background-color: limegreen;
	}
	#sniped-text {
		color: darkgreen;
	}
	#sniped-score {
		background-color: lightgreen;
	}
	
	#missed {
		background-color: red;
	}
	#missed-text {
		color: darkred;
	}
	#missed-score {
		background-color: #FFCCCB;
	}
	
	#rem {
		background-color: gray;
	}
	#rem-text {
		color: #dddddd;
	}
	#rem-score {
		background-color: lightgrey;
	}
</style>