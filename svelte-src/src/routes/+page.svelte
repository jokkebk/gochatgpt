<script>
import { onMount } from 'svelte';
import { marked } from 'marked';

let ai_types = [
    {name: "Coder", value: "You are a programming expert giving advice to a colleague."},
    {name: "Empty", value: ""},
];
let ai_type = ai_types[0].value;
let chat = [], question = '', waiting = false;
let textareaRef;

onMount(() => { textareaRef.focus(); });

function newChat() {
    chat = [];
    chat.push({role: 'system', content: ai_type});
}

function ask() {
    if(chat.length === 0) newChat();
    chat = [...chat, {role: 'user', content: question}];
    question = '';

    // Send the message over AJAX
    let xhr = new XMLHttpRequest();
    xhr.open("POST", "/chat", true);
    xhr.setRequestHeader('Content-Type', 'application/json');
    xhr.send(JSON.stringify(chat));
    waiting = true;
    xhr.onload = function() {
        if (xhr.status != 200) { // analyze HTTP status of the response
            alert(`Error ${xhr.status}: ${xhr.statusText}`); // e.g. 404: Not Found
        } else { // show the result
            chat = [...chat, {role: 'assistant', content: xhr.responseText}];
        }
        waiting = false; // clear the flag to allow further questions
        textareaRef.focus(); // focus on the textarea again
    };
}
</script>

<style>
    textarea {
        width: 100%;
        height: 100px;
        background-color: transparent;
        border: 1px solid #ccc;
    }
    
    div.system, div.assistant, div.user {
        padding: 10px;
        margin: 10px;
        border-radius: 10px;
    }

    div.system { background-color: #ccc; }
    div.assistant { background-color: #ccf; }
    div.user { background-color: #ffc; }
</style>

<h1>SvelteKit + Golang ChatGPT client demo</h1>

<p>
<select bind:value={ai_type}>
    {#each ai_types as ai_type}
        <option value={ai_type.value}>{ai_type.name}: {ai_type.value}</option>
    {/each}
</select>
<button on:click={newChat}>Start a new chat</button>
</p>

{#each chat as message}
    <div class={message.role}>
        {@html marked(message.content)}
    </div>
{/each}

{#if waiting}
    <div class="system">Waiting for the answer...</div>
{:else}
    <div class="user">
        <textarea bind:value={question} bind:this={textareaRef}
        on:keypress={e => { if(e.keyCode === 13 && e.ctrlKey) ask(); } }
        placeholder="Type your question here"></textarea>
        <button on:click={ask}>Ask</button>
    </div>
{/if}