<template>
  <section class="container">
    <div>
      <div class="all-surah">
        <Surah :surah-array="allSurahList" source="amp" />
      </div>

      <SeoText :paragraph="`Baca Juz Amma (Qur'an Juz 30) beserta terjemahan bahasa Indonesia dan tafsir dari Kemenag. Langsung dari peramban, tanpa iklan, tanpa analitik, privasi aman dan gratis sepenuhnya.`" />
    </div>
  </section>
</template>

<script lang="ts">
import { Component, Vue } from 'nuxt-property-decorator'
import { State, Mutation } from 'vuex-class'

import Surah from '~/components/Surah.vue'
import { AppConstant, META_TITLE_JUZ_AMMA, META_DESC_JUZ_AMMA } from '~/constant/index'
import { SurahInfo, SurahInfoJson } from '~/models/SurahInfo'

import SeoText from '~/components/SeoText.vue'

@Component({
  components: {
    Surah,
    SeoText
  },
  async asyncData () {
    // @ts-ignore
    const resp: SurahInfoJson = await import('~/data/surah-info.json')
    const juzAmmaData: SurahInfo[] = (resp && resp.surah_info.length > 0) ? resp.surah_info.filter(i => i.index >= 78) : []

    return {
      allSurahList: juzAmmaData.map((item) => {
        return Object.assign({}, item, { index: item.index })
      })
    }
  }
})

export default class PageJuzAmma extends Vue {
  @State settingActiveTheme
  @Mutation setHeaderTitle
  @Mutation setPage

  get metaHead () {
    return {
      title: META_TITLE_JUZ_AMMA,
      meta: [
        { hid: 'description', name: 'description', content: META_DESC_JUZ_AMMA },
        { hid: 'og:title', property: 'og:title', content: META_TITLE_JUZ_AMMA },
        { hid: 'twitter:title', name: 'twitter:title', content: META_TITLE_JUZ_AMMA },
        { hid: 'theme-color', name: 'theme-color', content: this.settingActiveTheme.bgColor }
      ],
      link: [
        { rel: 'canonical', href: `${AppConstant.PATH}juz-amma/` }
      ]
    }
  }

  head () {
    return this.metaHead
  }

  beforeMount () {
    this.setHeaderTitle('Juz Amma')
    this.setPage('juz-amma')
  }
}
</script>

<style lang="scss" scoped>
.all-surah{
  width: 90%;
  margin: 0 auto;
}
</style>
