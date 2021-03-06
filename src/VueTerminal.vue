<template>
  <div class="vue-terminal-wrapper">
    <div class="lds-css" v-if="this.waiting">
      <div style="width:100%;height:100%" class="lds-rolling"><div>
      </div>
    </div>
    </div>
      <div v-bind:style="{ maxHeight: this.height }" id="terminal" class="boring" data-theme="boring"
          v-bind:class="{ 'default-height': !fullScreen, 'fullscreen-height': fullScreen }"></div>
  </div>
</template>

<script>
    import './ptty.jquery.js'
    import $ from 'jquery'

    export default {
        name: 'VueTerminal',
        data: function () {
            return {
                waiting: false,
                ptty: null
            }
        },
        props: {
            height: { type: String, default: '100%' },
            intro: { type: String, default: '' },
            allowArbitrary: { type: Boolean, default: false },
            fullScreen: { type: Boolean, default: false },
            consoleSign: { type: String, default: '$' },
            write: { type: String, default: '' },
            show: { type: Boolean, default: false }
        },
        created() {
            if (this.show) this.create();
        },
        methods: {
            toggleWaiting() {
                this.waiting = !this.waiting;
            },
            toggleShow() {
                this.show = !this.show;
            },
            commandEmitter(commandText) {
                const prms = new Promise((resolve, reject) => {
                    const data = { text: commandText }
                    this.$emit('command', data, resolve, reject)
                    this.toggleWaiting()
                });

                prms.finally(this.toggleWaiting)
                return prms
            },

            create() {
                this.ptty = $('#terminal', '.vue-terminal-wrapper').Ptty({
                    i18n: { welcome: this.intro },
                    ps: this.consoleSign,
                    allowArbitrary: this.allowArbitrary,
                    passCommand: this.allowArbitrary ? this.commandEmitter : null
                });
            }
        },
        watch: {
            write(value) {
                if (value !== '' && this.ptty) this.ptty.echo(value);
            },
            show(value) {
                if (value) this.create();
                else this.ptty = null;
            },
            consoleSign(value) {
                this.ptty.change_settings({ ps: value });
            }
        }
    }
</script>

<style lang="css">
/* hight settings */
#terminal.default-height {
    max-height: 250px;
}
#terminal.fullscreen-height {
    height: 100vh;
}

/* basic bw theme */
.boring,
.boring .prompt,
.boring .content {
    font-family: "Courier New", Courier, monospace;
    background-color: #111;
    color: #ddd;
}
.boring .content {
    padding: 15px 15px 0 15px;
}
.boring .prompt {
    padding: 0 15px 15px 15px;
}
.boring .loading span::after {
    content: "⚙";
    color: #ddd;
    font-size: 10em;
    border-radius: 10em;
    opacity: 0.4;
}
.boring .content ul {
    margin: 0;
}
.boring .prompt .input.show-caret {
    color: #ddd;
    opacity: 0.85;
}
.boring .prompt .input,
.boring .prompt .input::before,
.boring .prompt .input::after {
    color: transparent;
    text-shadow: 0 0 0 #ddd;
}
.boring .content div .cmd_in .cmd_ps,
.boring .prompt .input::before {
    padding-right: 10px;
}
.boring .content ul li {
    list-style-type: none;
}
.boring div.prompt div.input::after {
    font-size: 2em;
}
.boring div.prompt div.input,
.boring div.content div div.cmd_in,
.boring div.prompt div.input::before {
    line-height: 2em;
}

/* other styles */
#terminal {
    position: relative;
    display: block;
    overflow-x: hidden;
    height: 100%;
    text-align: left;
}
#terminal div.content div p {
    margin: 0;
}
#terminal div.content div {
    clear: both;
    word-wrap: break-word;
}
#terminal div.content div ul {
    padding: 0;
    white-space: normal;
}
#terminal div.content div ul li {
    list-style: none;
}
#terminal div.content div ul.sq-li li {
    display: inline-block;
    text-align: center;
    padding: 10px;
    min-width: 5%;
}
#terminal div.prompt div.input {
    width: 100%;
    white-space: pre-wrap;
    word-wrap: break-word;
    cursor: default;
    outline: none;
}
#terminal div.prompt div.input::before {
    vertical-align: middle;
    content: attr(data-ps);
}
#terminal div.prompt div.input::after {
    visibility: visible;
    vertical-align: middle;
    content: attr(data-caret);
}
#terminal div.prompt div.input.blink::after {
    visibility: hidden;
}
#terminal div.prompt .hide {
    position: absolute;
    top: -9999em;
}
#terminal div.loading {
    display: none;
}
#terminal div.loading.working {
    display: block;
    display: flex;
    justify-content: center;
    align-items: center;
    position: fixed;
    width: inherit;
    height: inherit;
}
#terminal div.loading span {
    -webkit-animation: spin 4s linear infinite;
    -moz-animation: spin 4s linear infinite;
    -ms-animation: spin 4s linear infinite;
    -o-animation: spin 4s linear infinite;
    animation: spin 4s linear infinite;
}
@-moz-keyframes spin {
    100% {
        -moz-transform: rotate(360deg);
    }
}
@-webkit-keyframes spin {
    100% {
        -webkit-transform: rotate(360deg);
    }
}
@-ms-keyframes spin {
    100% {
        -ms-transform: rotate(360deg);
    }
}
@-o-keyframes spin {
    100% {
        -o-transform: rotate(360deg);
    }
}
@keyframes spin {
    100% {
        transform: rotate(360deg);
    }
}

/* loader */
@keyframes lds-rolling {
    0% {
        -webkit-transform: translate(-50%, -50%) rotate(0deg);
        transform: translate(-50%, -50%) rotate(0deg);
    }
    100% {
        -webkit-transform: translate(-50%, -50%) rotate(360deg);
        transform: translate(-50%, -50%) rotate(360deg);
    }
}
@-webkit-keyframes lds-rolling {
    0% {
        -webkit-transform: translate(-50%, -50%) rotate(0deg);
        transform: translate(-50%, -50%) rotate(0deg);
    }
    100% {
        -webkit-transform: translate(-50%, -50%) rotate(360deg);
        transform: translate(-50%, -50%) rotate(360deg);
    }
}
.lds-css {
    position: absolute;
}
.lds-rolling {
    position: fixed;
    top: 30px;
    z-index: 1000;
    right: 10px;
}
.lds-rolling div,
.lds-rolling div:after {
    position: absolute;
    width: 160px;
    height: 160px;
    border: 20px solid #ffffff;
    border-top-color: transparent;
    border-radius: 50%;
}
.lds-rolling div {
    -webkit-animation: lds-rolling 1s linear infinite;
    animation: lds-rolling 1s linear infinite;
    top: 100px;
    left: 100px;
}
.lds-rolling div:after {
    -webkit-transform: rotate(90deg);
    transform: rotate(90deg);
}
.lds-rolling {
    width: 36px !important;
    height: 36px !important;
    -webkit-transform: translate(-18px, -18px) scale(0.18) translate(18px, 18px);
    transform: translate(-18px, -18px) scale(0.18) translate(18px, 18px);
}
</style>
