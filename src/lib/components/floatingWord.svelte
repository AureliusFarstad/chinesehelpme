<script lang="ts">
	import { onMount, createEventDispatcher } from 'svelte';
	import { scale, fade, fly } from 'svelte/transition';
	import { linear } from 'svelte/easing'

    type HSKWord = {
        "simplified": string,
        "traditional": string,
        "pinyin": string,
        "definitions": string[];
    }

    // Declare the word, currentInput, and element props
	export let word: HSKWord;
	export let currentInput : string;
	let element : HTMLElement;
	let position : HTMLElement;
	
    // Declare state variables
	let visible: boolean = false;
    let showPinyin: boolean = false;
    let match: boolean = false;
	let bottomReached: boolean = false;

    let success: boolean = false;
	let failure: boolean = false;

	let maxY : number;
	let height : number;
	let bottom : number = 0;

    // Normalize the pinyin to match with user's currentInput
    const pinyin = word.pinyin.normalize("NFD").replace(/[\u0300-\u036f]/g, "")

	onMount(() => {
		const parentRect = position.parentElement!.getBoundingClientRect()
        // Initialize variables used to set and stop transitions
        maxY = parentRect.y + parentRect.height
		height = -1 * parentRect.height
        // Calculate and set a random x position
		const minX = parentRect.x + 16
		const maxX = parentRect.x + parentRect.width - 96
        console.log(minX)
        console.log(maxX)
        const x = Math.floor(Math.random() * (maxX-minX)) + minX;
        // Set the position.
		position.style.left = `${x}px`;
		position.style.bottom = `${0}px`;
        // Display the word triggering the in transition
		visible = true
	})
	
	const dispatch = createEventDispatcher();
	
	$: if (currentInput == pinyin) {
		if (!match && !bottomReached) {
			match = true;
            showPinyin = true;

			dispatch('match', {
				word: word,
				time: 1000
			})
            if (element.style) {
                element.style.animationPlayState = 'paused'
            }
			const rect = element.getBoundingClientRect()
			const currentY = rect.y + rect.height
			bottom = Math.round(maxY - currentY)

		}
	}
	
	function onBottomReached() {
        bottomReached=true
		showPinyin=true
		if (!match) {
			dispatch('miss', {
				word: word,
			})
		}
        if (element.style && position.style) {
            element.style.animationPlayState = 'play'
		    position.style.bottom = `${bottom}px`;
        }
	}
	
	function pinyinShowing() {
		if (match) {
			success = true
			setTimeout(() => {visible = false}, 200)
		} else {
			failure = true
			setTimeout(() => {visible = false}, 2000)
		}
	}
</script>

<div bind:this={position} class="position">
		{#if visible}
		<div
            bind:this="{element}"
            in:fly="{{y: height, duration: 30000, easing: linear, opacity: 1}}"
            on:introend="{onBottomReached}"
            out:fade="{{duration: 800}}"
            class:green="{match}"
            class:large="{success}"
            class:red="{failure}"
            class="word"
		>
            <!-- Hidden pinyin so width remains constant on pinyin display -->
            <div class="hidden">{word.pinyin}</div>
				{#if showPinyin}
				<div
						in:scale="{{duration: 1000}}"
						on:introend="{pinyinShowing}"
						class="match"
				>
						{word.pinyin}
				</div>
				{/if}
				{word.simplified}
		</div>
		{/if}
</div>

<style>
	.position {
		position: absolute;
        font-size: 32px;
        z-index: 9;
	}
    .hidden {
        color: transparent;
    }
	.word {
		width: 100%;
		text-align: center;
	}
	.green {
		color: limegreen;
		transition: color 0.5s, font-size 2s;
	}
	.large {
		font-size: 160px;
		color: red;
		transition: color 0.5s, font-size 2s;
	}
	.match {
		width: 100%;
		text-align: center;
		bottom: 16px;
	}
</style>