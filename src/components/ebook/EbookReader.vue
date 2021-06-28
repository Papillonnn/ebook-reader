<template>
  <div class="ebook-reader">
      <div id="read" />
  </div>
</template>

<script>
import { ebookMixin } from '@/utils/mixin'
import { saveFontFamily, getFontFamily, getFontSize, saveFontSize, getTheme, saveTheme } from '@/utils/localStorage'
import Epub from 'epubjs'
export default {
    mixins: [ebookMixin],
    mounted() {
        const fileName = this.$route.params.fileName.split('|').join('/')
        console.log(fileName)
        this.setFileName(fileName)
        this.initEpub()
    },
    methods: {
        initEpub() {
            // ComputerScience/2013_Book_AndroidOnX86.epub
            const bookUrl = `${process.env.VUE_APP_RES_URL}/epub/${this.fileName}.epub`
            this.book = new Epub(bookUrl)
            this.setCurrentBook(this.book)
            this.initRendition()
            this.initGesture()
            this.book.ready.then(() => {
                return this.book.locations.generate(750 * (window.innerWidth / 375) * (getFontSize(this.fileName) / 16))
            }).then(locations => {
                this.setBookAvailable(true)
            })
        },
        initRendition() {
            this.rendition = this.book.renderTo('read', {
                // flow: "paginated",
                // manager: "continuous",
                // snap: true,
                width: window.innerWidth,
                height: window.innerHeight,
                methods: 'default'
            })
            this.rendition.display().then(() => {
                this.initTheme()
                this.initFontSize()
                this.initFontFamily()
                this.initGlobalStyle()
            })
            // 添加字体文件
            this.rendition.hooks.content.register(contents => {
                contents.addStylesheet(`${process.env.VUE_APP_RES_URL}/fonts/cabin.css`)
                contents.addStylesheet(`${process.env.VUE_APP_RES_URL}/fonts/daysOne.css`)
                contents.addStylesheet(`${process.env.VUE_APP_RES_URL}/fonts/montserrat.css`)
                contents.addStylesheet(`${process.env.VUE_APP_RES_URL}/fonts/tangerine.css`)
            })
        },
        initGesture() {
            this.rendition.on('touchstart', event => {
                this.startX = event.changedTouches[0].clientX
                this.startTime = event.timeStamp
            })
            this.rendition.on('touchend', event => {
                const offsetX = event.changedTouches[0].clientX - this.startX
                const time = event.timeStamp - this.startTime
                if(time < 500 & offsetX > 40) {
                    this.prevPage()
                }else if(time < 500 & offsetX < -40) {
                    this.nextPage()
                }else {
                    this.toogleTitleAndMenu()
                }
                // event.preventDefault()
                event.stopPropagation()
            })
        },
        prevPage() {
            if(this.rendition) {
                this.rendition.prev()
                this.hideTitleAndMenu()
            }
        },
        nextPage() {
            if(this.rendition) {
                this.rendition.next()
                this.hideTitleAndMenu()
            }
        },
        toogleTitleAndMenu() {
            if(this.menuVisible) {
                this.setSettingVisible(-1)
            }
            this.setMenuVisible(!this.menuVisible)
            this.setFontFamilyVisible(false)
        },
        hideTitleAndMenu() {
            this.setMenuVisible(false)
            this.setSettingVisible(-1)
            this.setFontFamilyVisible(false)
        },
        initTheme() {
            let defaultTheme = getTheme(this.fileName)
            if(!defaultTheme) {
                defaultTheme = this.themeList[0].name
                saveTheme(this.fileName, defaultTheme)
            }
            this.setDefaultTheme(defaultTheme)
            this.themeList.forEach(theme => {
                this.rendition.themes.register(theme.name, theme.style)
            })
            this.rendition.themes.select(defaultTheme)
        },
        initFontSize() {
            const fontSize = getFontSize(this.fileName)
            if(!fontSize) {
                saveFontSize(this.fileName, this.defaultFontSize)
            }else {
                this.currentBook.rendition.themes.default({ "p": { "font-size": `${fontSize}px !important` } })
                this.setDefaultFontSize(fontSize)
            }
        },
        initFontFamily() {
            const fontFamily = getFontFamily(this.fileName)
            if(!fontFamily) {
                saveFontFamily(this.fileName, this.defaultFontFamily)
            }else {
                this.rendition.themes.font(fontFamily)
                this.setDefaultFontFamily(fontFamily)
            }
        },
        initGlobalStyle() {
            this.switchTheme(this.defaultTheme)
        }
    }
}
</script>

<style>

</style>
