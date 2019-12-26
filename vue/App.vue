<template>
    <div class='wrapper'>
        <main class='main'>
            <header class='header' :class='{ "header_collapsed" : currentStep !== STEPS.zero }'>
                <div class='container header__container'>
                    <a href='#' @click.prevent='setDefault'>
                        <h2 class='header__subtitle'>
                            <img src='../img/audio_to_midi.svg' alt='Audio To MIDI Converter' class='header__logo'>
                        </h2>
                        <h1 class='header__title'>
                            <img src='../img/AUDIOTOMIDI.svg' alt='AUDIO TO MIDI' class='header__title-logo'>
                        </h1>
                    </a>
                </div>
            </header>

            <div class='zero'
                 v-show='currentStep === STEPS.zero'
                 @click='start'>
                <h2 class='zero__title container'>
                    // Преобразование аудиофайлов в midi формат
                </h2>
                <div class='zero__start container'>
                    <img src='../img/mouse.svg' alt='Курсор' class='zero__cursor'>
                    <p class='zero__text'>Нажмите, чтобы начать</p>
                </div>
            </div>

            <form ref='form'
                  novalidate
                  v-show='currentStep !== STEPS.zero'
                  :class='{ "drop-form_recording" : currentStep === STEPS.recording }'
                  class='drop-form'>

                <input ref='file'
                       id='drop-form__file'
                       class='drop-form__file-input'
                       accept='audio/*'
                       type='file'
                       @change='onChangeInput'>

                <div class='drop-form__content container'>
                    <div class='drop-form__rec-container'>
                        <button type='button' class='drop-form__rec' v-if='!hasReturnButton' @click='onRecClick'>
                            <div class='drop-form__rec-image'>
                                <svg width="30" height="48" viewBox="0 0 30 48" fill="currentColor"
                                     xmlns="http://www.w3.org/2000/svg">
                                    <path
                                        d="M28.5124 16.9512C27.6907 16.9512 27.0248 17.6175 27.0248 18.4396V24.889C27.0248 31.5226 21.6302 36.9198 15 36.9198C8.36975 36.9198 2.97521 31.5226 2.97521 24.889V18.4396C2.97521 17.6175 2.30926 16.9512 1.4876 16.9512C0.66595 16.9512 0 17.6175 0 18.4396V24.889C0 32.6621 5.93752 39.0729 13.5124 39.8231V45.0233H6.75124C5.92959 45.0233 5.26364 45.6901 5.26364 46.5117C5.26364 47.3342 5.92959 48 6.75124 48H23.2488C24.0704 48 24.7364 47.3342 24.7364 46.5117C24.7364 45.6901 24.0704 45.0233 23.2488 45.0233H16.4876V39.8226C24.0625 39.0724 30 32.6617 30 24.8885V18.4391C30 17.617 29.334 16.9512 28.5124 16.9512ZM15 34.2547C20.2745 34.2547 24.5509 29.9757 24.5509 24.6985V9.55565C24.5509 4.27849 20.2745 0 15 0C9.72545 0 5.44909 4.27849 5.44909 9.55565V24.6985C5.44909 29.9757 9.72545 34.2547 15 34.2547Z"
                                    />
                                </svg>
                            </div>
                            <div class='drop-form__rec-text'>
                                Нажмите для записи с микрофона
                            </div>
                        </button>
                    </div>

                    <label for='drop-form__file' class='drop-form__label' :class='{ "drop-form__label_disabled" : dropDisabled }'
                           @click='onLabelClick'
                           @drop='onDrop'
                           @dragenter='onDragEnter'
                           @dragleave='onDragLeave'
                           @dragexit='onDragExit'
                           @dragover='onDragOver'>
                        <div class='drop-form__step'>
                            <transition name='fade'>
                                <div class='drop-form__step-content' v-if='currentStep === STEPS.index' key='index'>
                                    Перетащите или выберите аудиофайл
                                </div>

                                <div class='drop-form__step-content' v-if='currentStep === STEPS.dragover'
                                     key='dragover'>
                                    Отпускайте
                                </div>

                                <div class='drop-form__step-content' v-if='currentStep === STEPS.recording'
                                     key='recording'>
                                    Нажмите куда угодно, когда будете готовы
                                </div>

                                <div class='drop-form__step-content' v-if='currentStep === STEPS.incorrectFormat'
                                     key='incorrectFormat'>
                                    Мы такое не принимаем
                                </div>

                                <div class='drop-form__step-content' v-if='currentStep === STEPS.uploading'
                                     key='uploading'>
                                    Загружаем
                                    <br>
                                    <progress class='drop-form__progress'>
                                        {{ progress }}%
                                    </progress>
                                </div>

                                <div class='drop-form__step-content' v-if='currentStep === STEPS.uploadingError'
                                     key='uploadingError'>
                                    Что-то пошло не так при загрузке :(
                                </div>

                                <div class='drop-form__step-content' v-if='currentStep === STEPS.uploadingAbort'
                                     key='uploadingAbort'>
                                    Загрузка отменена
                                </div>

                                <div class='drop-form__step-content' v-if='currentStep === STEPS.converting'
                                     key='converting'>
                                    Конвертируем
                                    <br>
                                    <svg height="40" viewBox="0 0 135 140" xmlns="http://www.w3.org/2000/svg"
                                         fill="currentColor">
                                        <rect y="10" width="15" height="120" rx="6">
                                            <animate attributeName="height"
                                                     begin="0.5s" dur="1s"
                                                     values="120;110;100;90;80;70;60;50;40;140;120"
                                                     calcMode="linear"
                                                     repeatCount="indefinite"/>
                                            <animate attributeName="y"
                                                     begin="0.5s" dur="1s"
                                                     values="10;15;20;25;30;35;40;45;50;0;10" calcMode="linear"
                                                     repeatCount="indefinite"/>
                                        </rect>
                                        <rect x="30" y="10" width="15" height="120" rx="6">
                                            <animate attributeName="height"
                                                     begin="0.25s" dur="1s"
                                                     values="120;110;100;90;80;70;60;50;40;140;120"
                                                     calcMode="linear"
                                                     repeatCount="indefinite"/>
                                            <animate attributeName="y"
                                                     begin="0.25s" dur="1s"
                                                     values="10;15;20;25;30;35;40;45;50;0;10" calcMode="linear"
                                                     repeatCount="indefinite"/>
                                        </rect>
                                        <rect x="60" width="15" height="140" rx="6">
                                            <animate attributeName="height"
                                                     begin="0s" dur="1s"
                                                     values="120;110;100;90;80;70;60;50;40;140;120"
                                                     calcMode="linear"
                                                     repeatCount="indefinite"/>
                                            <animate attributeName="y"
                                                     begin="0s" dur="1s"
                                                     values="10;15;20;25;30;35;40;45;50;0;10" calcMode="linear"
                                                     repeatCount="indefinite"/>
                                        </rect>
                                        <rect x="90" y="10" width="15" height="120" rx="6">
                                            <animate attributeName="height"
                                                     begin="0.25s" dur="1s"
                                                     values="120;110;100;90;80;70;60;50;40;140;120"
                                                     calcMode="linear"
                                                     repeatCount="indefinite"/>
                                            <animate attributeName="y"
                                                     begin="0.25s" dur="1s"
                                                     values="10;15;20;25;30;35;40;45;50;0;10" calcMode="linear"
                                                     repeatCount="indefinite"/>
                                        </rect>
                                        <rect x="120" y="10" width="15" height="120" rx="6">
                                            <animate attributeName="height"
                                                     begin="0.5s" dur="1s"
                                                     values="120;110;100;90;80;70;60;50;40;140;120"
                                                     calcMode="linear"
                                                     repeatCount="indefinite"/>
                                            <animate attributeName="y"
                                                     begin="0.5s" dur="1s"
                                                     values="10;15;20;25;30;35;40;45;50;0;10" calcMode="linear"
                                                     repeatCount="indefinite"/>
                                        </rect>
                                    </svg>
                                </div>

                                <div class='drop-form__step-content' v-if='currentStep === STEPS.convertingError'
                                     key='convertingError'>
                                    Что-то пошло не так при конвертации :(
                                </div>

                                <div class='drop-form__step-content' v-if='currentStep === STEPS.ready' key='ready'>
                                    <a class='drop-form__link' :href='midiUrl'>Ваш MIDI-файл готов!</a>
                                </div>
                            </transition>
                        </div>
                    </label>
                </div>
            </form>
        </main>
        <footer class='footer'>
            <h3 class='footer__copyright'>Developed by Vyacheslav Majorov | 2019</h3>
        </footer>
    </div>
</template>
<script>
    const STEPS = {
        uploadingError: -1,
        uploadingAbort: -2,
        convertingError: -3,
        incorrectFormat: -4,
        zero: 0,
        index: 1,
        dragover: 2,
        uploading: 3,
        converting: 4,
        ready: 5,
        recording: 6,
    }
    export default {
        data() {
            return {
                STEPS,
                currentStep: STEPS.zero,
                file: null,
                progress: 0,
                midiUrl: null,
            }
        },
        mounted() {
            ['drag', 'dragstart', 'dragend', 'dragover', 'dragenter', 'dragleave', 'drop'].forEach(function (event) {
                this.$refs.form.addEventListener(event, function (e) {
                    e.preventDefault()
                    e.stopPropagation()
                }.bind(this), false)
            }.bind(this))
        },
        computed: {
            dropDisabled() {
                return ![STEPS.index, STEPS.dragover, STEPS.incorrectFormat].includes(this.currentStep)
            },
            hasReturnButton() {
                return this.dropDisabled && this.currentStep !== STEPS.recording
            },
        },
        methods: {
            start() {
                this.currentStep = STEPS.index
            },
            onRecClick() {
                if (this.currentStep !== STEPS.recording) this.currentStep = STEPS.recording
                else this.currentStep = STEPS.index
            },
            onLabelClick(e) {
                if (this.dropDisabled) e.preventDefault()
            },
            onDragEnter(e) {
            },
            onDragLeave() {
                this.currentStep = STEPS.index
            },
            onDragOver() {
                this.currentStep = STEPS.dragover
            },
            onDragExit() {
                this.currentStep = STEPS.index
            },
            onDrop(e) {
                this.validateAndSubmit(e.dataTransfer.files)
            },
            onUploading(e) {
                this.progress = Math.round(e.loaded / e.total * 100)
                if (e.loaded === e.total) this.currentStep = STEPS.converting
            },
            onComplete(e) {
                this.currentStep = STEPS.ready
                this.midiUrl = JSON.parse(e.target.responseText)['url']
                this.progress = 0

                window.open(this.midiUrl, '_self')
            },
            onReadyStateChange(e) {
                if (e.target.status !== 200 && e.target.readyState === 4) this.currentStep = STEPS.convertingError
            },
            onError() {
                if (this.progress !== 100) this.currentStep = STEPS.uploadingError
                else this.currentStep = STEPS.convertingError
            },
            onUploadingAbort() {
                this.currentStep = STEPS.uploadingAbort
            },
            setDefault() {
                this.currentStep = STEPS.zero
                this.file = null
                this.midiUrl = null
                this.progress = 0
                this.$refs.form.reset()
            },
            onChangeInput() {
                this.validateAndSubmit(this.$refs.file.files)
            },
            validateAndSubmit(files) {
                if (files.length !== 1 || !/^audio\/*/.test(files[0].type)) {
                    return this.currentStep = STEPS.incorrectFormat
                }
                this.file = files[0]
                this.submit()
            },
            submit() {
                this.currentStep = STEPS.uploading

                let formData = new FormData()
                formData.append('file', this.file)

                let xhr = new XMLHttpRequest()
                xhr.upload.addEventListener('progress', this.onUploading)
                xhr.addEventListener('load', this.onComplete)
                xhr.addEventListener('error', this.onError)
                xhr.addEventListener('abort', this.onUploadingAbort)
                xhr.addEventListener('readystatechange', this.onReadyStateChange)
                xhr.open('POST', `http://127.0.0.1:8000/audio_to_midi/convert`)
                xhr.send(formData)
            },
        },
    }
</script>
<style lang='sass'>
    \:root
        --gutter: 80px
        --text-color: hsl(0, 0%, 13%)
        --header-background-color: hsl(292, 90%, 40%)
        --header-color: white
        --form-background-color: white
        --background-color: hsl(292, 72%, 27%)
        --shadow-color: hsl(292, 80%, 21%)
        --accent-color: hsl(50, 94%, 55%)
        --accent-color-light: hsl(50, 100%, 66%)
        --warning-color: hsl(11, 94%, 55%)
        --warning-color-light: hsl(11, 98%, 64%)
        --muted-color: hsl(0, 0%, 53%)
        --link-color: var(--background-color)

        body
            color: var(--text-color)
            background-color: var(--background-color)

        .container
            max-width: 1000px
            padding: 0 20px
            margin: 0 auto
            width: 100%

        .wrapper
            height: 100vh
            width: 100vw
            display: flex
            flex-direction: column
            padding: var(--gutter) var(--gutter) 0

        .header
            --header-height: 300px
            --header-padding: 25px 0
            --logo-height: 73px
            --title-logo-height: 36px
            --subtitle-margin: 40px
            height: var(--header-height)
            background-color: var(--header-background-color)
            color: var(--header-color)
            padding: var(--header-padding)
            transition-duration: 0.2s
            transition-property: padding, height

            &_collapsed
                --header-height: 90px
                --header-padding: 15px 0
                --logo-height: 24px
                --title-logo-height: 12px
                --subtitle-margin: 0

            &__container
                display: flex
                flex-direction: column
                justify-content: flex-end
                height: 100%

            &__logo
                height: var(--logo-height)
                transition: height 0.2s

            &__title-logo
                height: var(--title-logo-height)
                transition: height 0.2s

            &__subtitle
                margin: 0 0 var(--subtitle-margin)
                transition: margin-bottom 0.2s
                line-height: 1

            &__title
                margin: 0
                line-height: 1

        .main
            flex: 1
            display: flex
            flex-direction: column
            background-color: var(--form-background-color)
            box-shadow: 0 4px 30px var(--shadow-color)

        .footer
            height: var(--gutter)
            display: flex
            align-items: center
            justify-content: center

            &__copyright
                font-size: 1rem
                font-weight: normal
                text-align: center
                color: white

        .zero
            cursor: pointer
            display: flex
            flex-direction: column
            flex: 1
            padding-top: 30px
            padding-bottom: var(--gutter)

            &__title,
            &__text
                font-weight: bold
                font-size: 24px
                line-height: 27px
                letter-spacing: 0.215em

            &__start
                margin-top: auto
                text-align: center

            &__text
                color: var(--muted-color)

            &__cursor
                transform-origin: 0 0
                animation: cursor-clicking 1s infinite

                @keyframes cursor-clicking
                    0%
                        transform: scale(1)
                    50%
                        transform: scale(0.9)
                    100%
                        transform: scale(1)

        .drop-form
            --form-color: var(--text-color)
            color: var(--form-color)
            height: 100%
            width: 100%
            flex: 1

            &_recording
                --accent-color: var(--warning-color)
                --accent-color-light: var(--warning-color-light)
                --form-color: white

            &__label
                position: relative
                flex: 1
                background-color: var(--accent-color)
                transition: background-color 0.2s, color 0.2s

                &_disabled,
                *:not(a):not(button)
                    pointer-events: none

                a, button
                    pointer-events: all

            &__file-input
                position: absolute
                visibility: hidden

            &__content
                display: flex
                flex-direction: column
                height: 100%

            &__rec-container
                padding-top: 32px
                padding-bottom: 64px
                height: 186px

            &__rec
                height: 90px
                text-align: center
                position: relative
                transition: transform 0.2s, background-color 0.2s, color 0.2s
                display: flex
                align-items: center
                padding: 0
                background-color: var(--accent-color)
                border-radius: 2px
                margin: 0 auto

                &:hover
                    transform: scale(1.1)

                &:active
                    transform: scale(1)

                &:focus
                    outline: none

            &__rec-image
                height: 90px
                width: 90px
                padding: 25px
                text-align: center
                background-color: var(--accent-color-light)
                transition: background-color 0.2s

                svg
                    height: 40px

            &__rec-text
                padding: 0 20px
                font-size: 22px


            &__border


            &__return
                font-size: 24px

            &__step
                font-size: 32px
                font-weight: bold

            &__step-content
                text-align: center
                position: absolute
                left: 50%
                top: 50%
                width: 100%
                transform: translate(-50%, -50%)

            &__link
                position: relative
                display: inline-block
                text-decoration: none
                color: var(--link-color)

                &::before
                    content: ''
                    display: block
                    height: 3px
                    width: 100%
                    position: absolute
                    bottom: -3px
                    transition: transform 0.2s
                    transform-origin: center
                    background-color: currentColor

                &:hover
                    color: var(--link-color)

                    &::before
                        transform: translateY(3px)

            .fade-enter-active, .fade-leave-active
                transition: transform 0.1s, opacity 0.1s

            .fade-enter, .fade-leave-to
                transform: translate(-50%, -50%) scale(0)
                opacity: 0
</style>
