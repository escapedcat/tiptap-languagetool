# tiptap-languagetool

Extension for integrating [Languagetool](https://languagetool.org/) with [TipTap](https://tiptap.dev). You can have your self-hosted instance of LanguageTool, details are [here](https://dev.languagetool.org/http-server). 

Special thanks to https://github.com/rezaffm for sponsoring this project.

https://user-images.githubusercontent.com/45892659/148092446-86816377-82c7-40be-940f-fa37e4f5a972.mp4

## How to use

Copy the [languagetool.ts](src/components/extensions/languagetool.ts) or [languagetool.js](dist/languagetool.js) file in your project depending on whether you use TypeScript or not. Then import the extension from that file and give it to the TipTap.

```ts
import { LanguageTool, Match } from './extensions/languagetool'

const match = ref<Match>(null)

const updateMatch = (editor: Editor) => match.value = editor.extensionStorage.languagetool.match

const replacements = computed(() => match.value?.replacements || [])

const matchMessage = computed(() => match.value?.message || 'No Message')

const updateHtml = () => navigator.clipboard.writeText(editor.value.getHTML())

const acceptSuggestion = (sug) => {
  editor.value.commands.insertContent(sug.value)
}

const proofread = () => editor.value.commands.proofread()

const editor = useEditor({
  content,
  extensions: [StarterKit, LanguageTool.configure({ 
    language: 'auto', // it can detect language automatically or you can write your own language like 'en-US'
    apiUrl: "https://api.languagetool.org/v2/" + 'check', // See note below
    automaticMode: true, // if true, it will start proofreading immediately otherwise only when you execute `proofread` command of the extension.
  })],
  onUpdate({ editor }) {
    setTimeout(() => updateMatch(editor as any))
  },
  onSelectionUpdate({ editor }) {
    setTimeout(() => updateMatch(editor as any))
  },
})
```

# Language Tool URL Public HTTP Proofreading API 

Language tool offer a public API endpoint: `https://api.languagetool.org/v2/check`. When using this link you should be responsible as outlined here.

When using it, please keep the following [rules](https://dev.languagetool.org/public-http-api) in mind:
---
- Do not send automated requests. For that, set up your own instance of LanguageTool or get an account for Enterprise use.
- Only send POST requests, not GET requests.
- Access is currently limited to:
- 20 requests per IP per minute (this is supposed to be a peak value - don’t constantly send this many requests or we would have to block you)
- 75KB text per IP per minute
- 20KB text per request
- Only up to 30 misspelled words will have suggestions.
- This is a free service, thus there are no guarantees about performance or availability. The limits may change anytime.
- The LanguageTool version installed may be the latest official release or some snapshot. We will simply deploy new versions, thus the behavior will change without any warning.
- Read our privacy policy to see how we handle your texts. You are responsible for giving your users information about how their data is handled.
- We expect you to add a link back to https://languagetool.org that’s clearly visible.
---

Now showing the suggestion on click, so now in the vue component where you've implemented tiptap.

```vue
<bubble-menu
  class="bubble-menu"
  v-if="editor"
  :editor="editor"
  :tippy-options="{ placement: 'bottom', animation: 'fade' }"
>
  <section class="bubble-menu-section-container">
    <section class="message-section">
      {{ matchMessage }}
    </section>
    <section class="suggestions-section">
      <article
        v-for="(replacement, i) in replacements"
        @click="() => acceptSuggestion(replacement)"
        :key="i + replacement.value"
        class="suggestion"
      >
        {{ replacement.value }}
      </article>
    </section>
  </section>
</bubble-menu>
```

You can implement your own styles or copy the ones in [Tiptap.vue](src/components/Tiptap.vue).

-------------------------------------------------------------
-------------------------------------------------------------

## Stuff that nobody really cares about(Project setup)
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
