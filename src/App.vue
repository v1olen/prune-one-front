<template>
    <div class="View">
        <header class="View__Header">
            <transition name="fade">
                <h1
                    v-if="!createdUrl"
                    class="View__Title"
                >
                    prune.one
                </h1>
            </transition>
            <transition name="fade">
                <form
                    v-if="!createdUrl"
                    class="View__Board Form"
                    autocomplete="off"
                    @submit.prevent
                >
                    <input
                        v-if="!isSlugFieldVisible"
                        id="url"
                        key="url"
                        v-model="url"
                        class="Form__Input"
                        type="text"
                        name="input"
                        placeholder="Enter your URL"
                        @keydown.enter="nextStep"
                        @focus="setMessage(`PRESS_ENTER`)"
                        @blur="setMessage(`EMPTY`)"
                        @input="setMessage(`PRESS_ENTER`)"
                    >                    
                    <input
                        v-else
                        id="slug"
                        key="slug"
                        ref="slug"
                        v-model="slug"
                        class="Form__Input"
                        type="text"
                        name="input"
                        placeholder="Choose a unique slug"
                        @keydown.enter="nextStep"
                        @focus="setMessage(`PRESS_ENTER`)"
                        @blur="setMessage(`EMPTY`)"
                        @input="setMessage(`PRESS_ENTER`)"
                    >
                </form>
                <div v-else class="View__Board Result">
                    <span class="Result__Label">Your pruned URL is:</span>
                    <span class="Result__Url">{{ createdUrl }}</span>
                </div>
            </transition>
            <transition name="fade">
                <span class="View__Message">{{ message }}</span>
            </transition>
        </header>
        <main class="View__Content">
            <h2 class="View__Heading">
                What's this?
            </h2>
            <p class="View__Paragraph">
                A link shortener, built with Rust.
            </p>
            <h2 class="View__Heading">
                So… it just shortens links?
            </h2>
            <p class="View__Paragraph">
                Yeah, or <i>prunes</i> them, if you prefer. Paste the ugly URL you need to prune, set a custom slug and voila.
            </p>
            <h2 class="View__Heading">
                Where's your code?
            </h2>
            <p class="View__Paragraph">
                Take a look at the
                <a
                    href="https://github.com/TheViolence/prune-one"
                    class="Link"
                    target="_blank"
                >
                    prune.one repository on GitHub
                </a>.
            </p>
        </main>
    </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import axios, { AxiosResponse } from "axios";
import { CreatedBridge, NewBridge } from "./types";

@Component({
    components: {
    },
})
export default class App extends Vue {
    step = 0;
    isSlugFieldVisible = false;
    urlPattern = /[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&//=]*)/;
    protocolPattern = /^https?:\/\//;
    url = ``;
    slug = ``;
    createdUrl = ``;
    messages: {
        [key: string]: string;
    } = {
        INVALID_URL: `Please provide a valid URL`,
        PRESS_ENTER: `Hit enter when you're ready`,
        EMPTY: ``,
    };
    message = ``;

    async nextStep() {
        switch (this.step) {
        case 0: 
            if (!this.urlPattern.test(this.url)) {
                this.setMessage(`INVALID_URL`);
                return;
            }
            this.isSlugFieldVisible = true;
            // eslint-disable-next-line
            // @ts-ignore: Unreachable code error
            setTimeout(() => this.$refs.slug.focus(), 0);
            break;
        case 1:
            const url = this.protocolPattern.test(this.url) ? this.url : `http://${this.url}`;
            const data: NewBridge = {
                slug: this.slug,
                target_url: url,
            };
            try {
                const createdBridge: AxiosResponse<CreatedBridge> = await axios
                    .post(`/bridge`, data);
                this.createdUrl = createdBridge.data.url;
                this.setMessage(`EMPTY`);
            } catch (error) {
                console.log(error);
            }
            break;
        }

        if (this.step != 2) this.step++;
    }

    setMessage(type: string) {
        this.message = this.messages[type];
    }
}
</script>

<style lang="sass">
::selection 
    color: #1b4035
    background: #fff

.View
    *
        color: #cdcdcd
        font-family: "Noto Sans JP"
    &__Header
        height: 17rem
        background-color: #592631
        display: flex
        justify-content: center
        align-items: center
        flex-direction: column
        position: relative
    &__Title
        font-size: 3.5rem
    &__Content
        height: calc(100vh - 17rem)
        max-width: 55rem
        margin: 0 auto
        padding: 4rem 1rem
    &__Heading
        font-size: 3.5rem
        margin-bottom: 2rem
    &__Paragraph
        font-size: 1.5rem
        margin-bottom: 4rem
    &__Message 
        position: absolute
        bottom: 2.5rem
        color: rgba(255,255,255,.3)

.Form__Input
    height: 3.5rem
    width: 40rem
    margin: 2rem
    padding: 1rem
    font-size: 1.5rem
    background: rgba(255,255,255,.0666)
    border: 0
    border-radius: .5rem
    text-align: center
    &::placeholder
        color: #999
        text-align: center

.Result 
    position: absolute
    display: flex
    flex-direction: column
    align-items: center
    &__Url
        font-size: 3.5rem

</style>
