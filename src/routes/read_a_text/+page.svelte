<script lang="ts">
    import SegmentTable from './segmentTable.svelte';
    import xss from 'xss';
    const transformTextEndpoint = "https://marcusfarstad.com/transform/"

    type CEDICTEntry = {
        definitions: string[],
        pinyin: string
    }

    type Segment = {
        ws: string;
        pos: string[];
        def: CEDICTEntry[];
    }

    // The page has two states: 
    // editing = true -> edit the input text
    var editing = true;
    let textBoxEl: HTMLElement;

    // editing = false -> select segments from the text */
    var segments: Segment[] = []
    var toggledSegments: Segment[] = []
    $: toggledWords = toggledSegments.map(w => w.ws)
    function compareSegments(a: Segment, b: Segment) {
        var index_a = segments.findIndex((segment) => segment.ws == a.ws)
        var index_b = segments.findIndex((segment) => segment.ws == b.ws)
        return index_a - index_b
    }
    $: sortedSegments = [...toggledSegments].sort(compareSegments)
    $: sortedWords = sortedSegments.map(w => w.ws)

    function toggle(segment: Segment) {
        var index = toggledWords.indexOf(segment.ws);
        if (index === -1) {
            toggledSegments = [...toggledSegments, segment];
        } else {
            toggledSegments.splice(index, 1);
            toggledSegments = toggledSegments
        }
    }

    // Return to edit input text mode. todo: more logic on handover.
    async function editText() {
        editing = true
    }

    async function segmentChinese(sentences: string[]) {
        const body = {
            sentences: sentences
        }
        const headers = {
            'Content-Type':'application/json',
            'Access-Control-Allow-Origin':'*',
            'Access-Control-Allow-Methods':'POST,PATCH,OPTIONS',
            'Access-Control-Allow-Headings':'Content-Type',
        }
        // Make the request asynchronously using the `await` keyword
        const response = await fetch(transformTextEndpoint, {
            method: "POST",
            body: JSON.stringify(body),
            headers: headers
        })
        const data = await response.json()
        return data
    }

    function splitSentences(text: string) {
        const delimRegex = /[。.?？!！]/g;
        let splitText = text.split(delimRegex);
        let delimeters = text.match(delimRegex);
        const sentences = splitText.map((sentence, index) => `${sentence}${delimeters![index]||""}`);
        return sentences
    }

    async function submitText() {
        try {
            let text = textBoxEl.innerText
            if (text == "") {
                alert("Text is required.");
                return;
            }
            // Sanitize the text
            text = xss(text)
            
            // Split the text input into an array of sentences
            let sentences = splitSentences(text)
            // Remove any empty strings from the array of sentences
            sentences = sentences.filter(Boolean);
            console.log("fitler")
            console.log(sentences)
            if (sentences.length == 0) {
                alert("Text is required.");
                return;
            }            // Remove any leading or trailing whitespace from each sentence in the array
            sentences = sentences.map(sentence => sentence.trim());

            // Wait for the result of the `segmentChinese` function before continuing
            const data = await segmentChinese(sentences);
            // Enter toggling mode
            segments = data.segments
            editing = false
        } catch (error) {
            console.error(error);
            return error;
        }
    }

    function handlePaste(event: ClipboardEvent) {
        // Get the pasted text as a string
        let pastedText = event.clipboardData!.getData('text/plain');
        // Remove any unwanted styles from the pasted text
        pastedText = pastedText.replace(/style="[^"]*"/g, '');
        // Prevent the default paste behavior
        event.preventDefault();
        // Insert the modified text into the document
        document.execCommand('insertHTML', false, pastedText);
    }
</script>

<div class="on-screen">
{#if editing}
    <div 
        id="editable" class="box" 
        bind:this={textBoxEl} 
        contentEditable=true 
        data-text="写这里。Enter text here."
        on:paste={handlePaste}
    ></div>

    <div class="button-row">
        <button id="submit-text" class="button" on:click={submitText}>Submit text.</button>
    </div>

{:else}
    <div id="clickable" class="box">
        {#each segments as s}
            <!-- svelte-ignore a11y-click-events-have-key-events -->
            <span 
                on:click="{() => {toggle(s)}}"
                class="segmentSpan {toggledWords.includes(s.ws) ? 'selected': ''}"
            >{s.ws}{#if toggledWords.includes(s.ws)}<span class="footnote">{sortedWords.indexOf(s.ws)}</span>{/if}</span>
        {/each}
    </div>

    <div class="button-row">
        <button id="edit-text" class="button" on:click={editText}>Edit text.</button>
    </div>

    <SegmentTable {sortedSegments}/>
{/if}
</div>

<div id="container"></div>

<style>
    /* Because of rendering?... */
#container {
    position: absolute;
    z-index: 20;
    background-color: white;
    bottom: 0px;

    width: 100%;
    height: calc(100% - 50px);
}
.on-screen {
    display: flex;
    z-index: 30;
    flex-flow: column;

    background-color: white;

    width: 100%;
    max-width: 1020px;
    position: absolute;
    height: calc(100%-66px);

    left: 50%;
    top: 50px;
    bottom: 0px;
    transform: translate(-50%, 0);
 
    padding-top: 8px;
    padding-bottom: 8px;
    padding-left: 8px;
    padding-right: 8px;
}
.box {
    text-align: left;
    min-height: 50%;
    border-radius: 3px;

    padding-top: 15px;
    padding-left: 5px;
    padding-right: 5px;

    font-family: 'STSong', 'SimSun', 'NSimSun', serif;
    font-size: 18px;
    line-height: 48px;
    vertical-align: text-bottom;
    overflow: scroll;
}
#editable { 
    color: black;
    border: 1px solid #009900;
}
#clickable {
    background-color: rgb(222, 222, 222);
    color: black;
    border: 1px solid black;
}
.selected {
    color: #ce0000;
    border: 1px solid red;
}
.button {
    font-family: 'Futura', 'Nunito Sans', 'Calibri', 'Verdana', sans-serif;
    width: 150px;
    height: 40px;
    border: none;

    border-radius: 4px;
  
    font-size: 18px;
}
#submit-text {
    background-image: linear-gradient(60deg, #33CC33, #00CC00);
    /* border: 1px solid transparent; */
    color: white;
    transition: all 0.1s ease;
}
#submit-text:hover {
    /* border: 1px solid #009900; */
    background-image: linear-gradient(60deg, #66CC66, #00FF00);
    color: black;
}
.button-row {
    display: flex;
    margin-top: 8px;
}
#edit-text {	
  background-color: black;
  color: white;
  transition: all 0.3s ease;
}
#edit-text:hover {	
    background-image: linear-gradient(60deg, lightgray, gray);
  color: black;
}
[contentEditable=true]:empty:not(:focus):before {
  content: attr(data-text);
  color: #009900;
}





.segmentSpan {
    position: relative;
}
.footnote {
    font-size: 12px;
    position: absolute;
    bottom: 4px;
    left: 0px;
}

/* https://www.onlinewebfonts.com/download/456f6968af7fdbde28970967bfdc4d12 */
</style>
