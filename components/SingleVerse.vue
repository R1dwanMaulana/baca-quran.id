<template>
  <div :id="`verse-${verseIndex}`" class="verse block_content has-shadow" :data-source="source">
    <div class="verse__header">
      <div class="verse__index tag_index">
        {{ Number(verseIndex) }}
      </div>
      <div class="verse__header--right">
        <div v-if="!isAmp" class="verse__header_icon" @click="onClickAudioItem(Number(verseIndex))">
          <MdVolumeHighIcon w="2em" h="2em" />
        </div>

        <div
          v-if="!isAmp"
          class="verse__header_icon"
          @click="doSetLastReadVerse({ surah: surahId, verse: Number(verseIndex) })">
          <MdBookmarkIcon w="2em" h="2em" />
        </div>

        <div
          v-if="!isAmp && isSupportWebShare"
          class="verse__header_icon"
          @click="shareVerse(verse, Number(verseIndex))">
          <MdShareIcon w="2em" h="2em" />
        </div>

        <nuxt-link
          v-if="!isAmp && !source.includes('verse')"
          class="verse__header_icon"
          :to="`/${surahId}/${verseIndex}/`">
          <MdLinkIcon w="2em" h="2em" />
        </nuxt-link>
      </div>
    </div>
    <div class="divider clearfix">
      <div class="verse__arabic font-arabic" dir="rtl" lang="ar">
        {{ verse.trim() }}
      </div>
    </div>

    <div v-if="!showSettings || settingShowTranslation" class="divider clearfix">
      <div class="verse__bold">
        <b>Terjemah:</b>
      </div>
      <div class="verse__translation">
        {{ getTranslation(verseIndex) }}
      </div>
    </div>

    <div v-if="!showSettings || settingShowTafsir" class="divider clearfix">
      <div class="verse__bold">
        <b>Tafsir dari Kemenag:</b>
      </div>
      <div class="verse__tafsir">
        {{ getTafsir(verseIndex) }}
      </div>
      <div class="verse__small">
        <i>Sumber: Aplikasi Quran Kementrian Agama Republik Indonesia</i>
      </div>
    </div>

    <audio v-if="!isAmp && audioLink" :id="`audioVerseRef${verseIndex}`" class="audio">
      <source :src="audioLink" type="audio/mpeg">
    </audio>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'nuxt-property-decorator'
import { State, Action } from 'vuex-class'

import MdShareIcon from 'vue-ionicons/dist/js/md-share'
import MdBookmarkIcon from 'vue-ionicons/dist/js/md-bookmark'
import MdVolumeHighIcon from 'vue-ionicons/dist/js/md-volume-high'
import MdLinkIcon from 'vue-ionicons/dist/js/md-link'
import MurotalConstant from '~/constant/murotal'
import { AppConstant } from '~/constant'

@Component({
  components: {
    MdShareIcon,
    MdBookmarkIcon,
    MdVolumeHighIcon,
    MdLinkIcon
  }
})
export default class SingleVerseCard extends Vue {
  AppConstant = AppConstant;
  audioLink = '';

  @Prop({ type: [Object, Array], default: () => ({}) }) readonly verseArray!:
    | any
    | any[];

  @Prop({ type: Object, default: () => ({}) }) readonly translations!: any;
  @Prop({ type: Object, default: () => ({}) }) readonly tafsir!: any;
  @Prop({ type: Number, default: 1 }) readonly surahId!: number;
  @Prop({ type: String, default: '' }) readonly verse!: string;
  @Prop({ type: String, default: '1' }) readonly verseIndex!: number;
  @Prop({ type: String, default: '' }) readonly source!: string;
  @Prop({ type: Boolean, default: false }) readonly showSettings!: boolean;

  @State surahFavorite;
  @State isSupportWebShare;
  @State settingShowTranslation;
  @State settingShowTafsir;

  @Action setLastReadVerse;
  @Action showNotification;
  @Action shareViaWebshare;

  get isAmp (): boolean {
    return this.source.includes('amp')
  }

  onClickAudioItem (verse) {
    const hrefAudio = MurotalConstant.getAudioFromKemenag(this.surahId, verse)
    this.audioLink = hrefAudio

    setTimeout(async () => {
      try {
        // @ts-ignore
        const node = document.querySelector(`#audioVerseRef${verse}`)
        // @ts-ignore
        await node.play()
      } catch {}
    }, 500)
  }

  getTranslation (indexVerse) {
    return this.translations.id.text[indexVerse]
  }

  getTafsir (indexVerse) {
    return this.tafsir.id.kemenag.text[indexVerse]
  }

  doSetLastReadVerse (data) {
    this.setLastReadVerse(data)
    this.showNotification({
      title: 'Pesan Sukses',
      message: `QS ${data.surah}:${data.verse} telah ditambahkan ke terakhir dibaca.`
    })
  }

  shareVerse (verse, index) {
    const data = {
      title: `QS ${this.surahId}:${index}`,
      text: `${verse}

        Terjemahan: ${this.getTranslation(index)} (QS ${
        this.surahId
      }:${index})`,
      url: `${AppConstant.PATH}${this.surahId}/${index}/`
    }
    this.shareViaWebshare(data)
  }
}
</script>

<style lang="scss" scoped>
.verse {
  text-align: left;

  &__header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    &--right {
      display: flex;
      align-items: center;
      justify-content: space-between;
    }
    &_icon {
      padding: 0 0.5em;
      position: relative;
      color: var(--text-color);
    }
  }
  &__arabic {
    font-size: 2rem;
    width: 100%;
    float: right;
    text-align: right;
    margin-top: 0.25em;
  }
  &__translation {
    font-size: 1rem;
    width: 100%;
    font-style: italic;
    margin-top: 1.5em;
  }
  &__tafsir {
    font-size: 1rem;
    width: 100%;
    margin-top: 1.5em;
  }
  &__bold {
    font-weight: bold;
    margin-top: 1.5em;
    line-height: 1.5;
  }
  &__small {
    font-size: 0.8rem;
    margin-top: 1.5em;
    font-style: italic;
  }
}
.audio {
  position: fixed;
  bottom: 70px;
}
</style>
