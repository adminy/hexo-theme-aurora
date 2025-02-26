<template>
  <div
    class="h-98 w-full rounded-2xl relative shadow-xl mb-8"
    :style="gradientBackground"
  >
    <div
      class="ob-gradient-cut-plate absolute bg-ob-deep-900 rounded-xl opacity-90 flex justify-center items-center pt-4 px-6 shadow-lg hover:shadow-2xl duration-300"
      data-dia="author"
    >
      <div
        class="profile absolute w-full flex flex-col justify-center items-center"
      >
        <div class="flex flex-col justify-center items-center">
          <img
            v-if="authorData.avatar !== ''"
            :class="avatarClass"
            :src="authorData.avatar"
            alt="avatar"
          />
          <ob-skeleton v-else width="7rem" height="7rem" circle />

          <h2 class="text-center pt-4 text-4xl font-semibold text-ob-bright">
            <template v-if="authorData.name">
              {{ authorData.name }}
            </template>
            <ob-skeleton v-else height="2.25rem" width="7rem" />
          </h2>

          <span
            class="h-1 w-14 rounded-full mt-2"
            :style="gradientBackground"
          />

          <p
            v-if="authorData.description"
            class="pt-6 px-10 w-full text-sm text-center"
            v-html="authorData.description"
          />
          <p
            v-else
            class="pt-6 px-10 w-full text-sm text-center flex flex-col gap-2"
          >
            <ob-skeleton :count="2" height="20px" width="10rem" />
          </p>
        </div>
        <div class="h-full w-full flex flex-col flex-1 justify-end items-end">
          <Social :socials="authorData.socials" />
          <ul class="grid grid-cols-4 pt-4 w-full px-2 text-lg">
            <li class="col-span-1 text-center">
              <span class="text-ob-bright">{{ authorData.word_count }}</span>
              <p class="text-base">{{ t('settings.words') }}</p>
            </li>
            <li class="col-span-1 text-center">
              <span class="text-ob-bright">
                {{ authorData.post_list.length }}
              </span>
              <p class="text-base">{{ t('settings.articles') }}</p>
            </li>
            <li class="col-span-1 text-center">
              <span class="text-ob-bright">{{ authorData.categories }}</span>
              <p class="text-base">{{ t('settings.categories') }}</p>
            </li>
            <li class="col-span-1 text-center">
              <span class="text-ob-bright">{{ authorData.tags }}</span>
              <p class="text-base">{{ t('settings.tags') }}</p>
            </li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { AuthorPosts } from '@/models/Post.class'
import { useAppStore } from '@/stores/app'
import { useAuthorStore } from '@/stores/author'
import { computed, defineComponent, onMounted, ref, toRefs, watch } from 'vue'
import { useI18n } from 'vue-i18n'
import Social from '@/components/Social.vue'

export default defineComponent({
  name: 'ObProfile',
  components: { Social },
  props: {
    author: {
      type: String,
      default: () => {
        return ''
      }
    }
  },
  setup(props) {
    const appStore = useAppStore()
    const authorStore = useAuthorStore()
    const { t } = useI18n()

    const author = toRefs(props).author
    const authorData = ref(new AuthorPosts())

    const fetchData = async () => {
      await appStore.fetchStat()
      await fetchAuthor()
    }

    const fetchAuthor = async () => {
      if (author.value === '') return
      const blogAuthor = appStore.themeConfig.site.author.replaceAll(' ', '-').toLowerCase()
      await authorStore.fetchAuthorData(blogAuthor).then(data => {
        authorData.value = data
      })
    }

    watch(
      () => props.author,
      (newAuthor, oldAuthor) => {
        if (newAuthor && newAuthor !== oldAuthor) {
          fetchAuthor()
        }
      }
    )

    onMounted(fetchData)

    return {
      avatarClass: computed(() => {
        return {
          'ob-avatar': true,
          [appStore.themeConfig.theme.profile_shape]: true
        }
      }),
      themeConfig: computed(() => appStore.themeConfig),
      gradientBackground: computed(() => {
        return { background: appStore.themeConfig.theme.header_gradient_css }
      }),
      socials: computed(() => {
        return appStore.themeConfig.socials
      }),
      statistic: computed(() => appStore.statistic),
      authorData,
      t
    }
  }
})
</script>

<style lang="scss" scoped>
.profile {
  top: -7%;
  height: 100%;
  max-height: 100%;
}
</style>
