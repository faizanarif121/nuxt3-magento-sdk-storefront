<template>
  <div class="w-full h-full">
    <div v-if="isOpen" class="fixed w-screen h-screen inset-0 bg-neutral-500 bg-opacity-50 transition-opacity" />
    <header ref="menuRef"
      class="flex justify-center w-full border-0 bg-primary-700 border-neutral-200 h-14 md:relative md:h-20 md:z-10">
      <div class="flex items-center flex-nowrap justify-start h-full max-w-[1536px] w-full px-4 md:px-10">
        <a href="/" aria-label="SF Homepage"
          class="flex shrink-0 text-white mr-2 md:mr-10 focus-visible:outline focus-visible:outline-offset focus-visible:rounded-sm">
          <picture>
            <source
              srcset="https://storage.googleapis.com/sfui_docs_artifacts_bucket_public/production/vsf_logo_white.svg"
              media="(min-width: 1024px)" />
            <img src="https://storage.googleapis.com/sfui_docs_artifacts_bucket_public/production/vsf_logo_sign_white.svg"
              alt="Sf Logo" class="w-8 h-8 lg:w-[12.5rem] lg:h-[1.75rem]" />
          </picture>
        </a>
        <nav class="flex w-full justify-between flex-nowrap" aria-label="SF Navigation">
          <ul>
            <li role="none">
              <SfButton
                class="block mr-auto text-white font-body bg-transparent hover:bg-primary-800 hover:text-white active:bg-primary-900 active:text-white"
                type="button" :aria-haspopup="true" :aria-expanded="isOpen" variant="tertiary" @click="toggle()">
                <template #suffix>
                  <SfIconExpandMore />
                </template>
                <span class="hidden md:inline-flex">Browse products</span>
              </SfButton>
              <transition enter-active-class="transform transition duration-500 ease-in-out"
                leave-active-class="transform transition duration-500 ease-in-out"
                enter-from-class="-translate-x-full md:translate-x-0 md:opacity-0"
                enter-to-class="translate-x-0 md:translate-x-0 md:opacity-100"
                leave-from-class="translate-x-0 md:opacity-100"
                leave-to-class="-translate-x-full md:translate-x-0 md:opacity-0">
                <SfDrawer ref="drawerRef" v-model="isOpen" disable-click-away placement="top"
                  class="grid grid-cols-1 md:gap-x-6 md:grid-cols-4 bg-white max-w-xs shadow-lg p-0 !fixed max-h-screen overflow-y-auto md:!absolute md:!top-[5rem] md:max-w-full md:p-6">
                  <div class="flex items-center justify-between py-2 px-4 bg-primary-700 md:hidden">
                    <div class="flex items-center typography-text-lg font-medium text-white">Browse products</div>
                    <SfButton square variant="tertiary" aria-label="Close navigation menu" class="text-white"
                      @click="close()" @keydown.enter.space="close()">
                      <SfIconClose />
                    </SfButton>
                  </div>
                  <div v-for="({ heading, items }, index) in categoriesContent" :key="index"
                    class="[&:nth-child(2)]:pt-0 pt-6 md:pt-0">
                    <h2 role="presentation"
                      class="typography-text-base font-medium text-neutral-900 whitespace-nowrap p-4 md:py-1.5">
                      {{ heading }}
                    </h2>
                    <hr class="mb-3.5" />
                    <ul>
                      <li v-for="item in items" :key="item.title">
                        <SfListItem tag="a" :href="item.link" size="sm" role="none"
                          class="typography-text-base md:typography-text-sm py-4 md:py-1.5">
                          {{ item.title }}
                        </SfListItem>
                      </li>
                    </ul>
                  </div>
                  <SfButton square size="sm" variant="tertiary" aria-label="Close navigation menu"
                    class="hidden md:block md:absolute md:right-0 hover:bg-white active:bg-white" @click="close()">
                    <SfIconClose class="text-neutral-500" />
                  </SfButton>
                </SfDrawer>
              </transition>
            </li>
          </ul>
          <div class="flex flex-nowrap">
            <SfButton v-for="actionItem in actionItems" :key="actionItem.ariaLabel"
              class="mr-2 -ml-0.5 text-white bg-transparent hover:bg-primary-800 hover:text-white active:bg-primary-900 active:text-white"
              :aria-label="actionItem.ariaLabel" variant="tertiary" square>
              <template #prefix>
                <Component :is="actionItem.icon" />
              </template>
              <span v-if="actionItem.role === 'login'" class="hidden md:inline-flex">{{ actionItem.label }}</span>
            </SfButton>
          </div>
        </nav>
      </div>
    </header>
  </div>
</template>
<script lang="ts" setup>
import {
  SfButton,
  SfDrawer,
  SfIconShoppingCart,
  SfIconFavorite,
  SfIconPerson,
  SfIconClose,
  SfIconExpandMore,
  SfListItem,
  useDisclosure,
  useTrapFocus,
} from '@storefront-ui/vue';
import { ref } from 'vue';
import { onClickOutside } from '@vueuse/core';
import { sdk } from "~/sdk.config";

const { isOpen, toggle, close } = useDisclosure();
const menuRef = ref();
const drawerRef = ref();

useTrapFocus(drawerRef, {
  activeState: isOpen,
  arrowKeysUpDown: true,
  initialFocus: 'container',
});
onClickOutside(menuRef, () => {
  close();
});

const actionItems = [
  {
    icon: SfIconShoppingCart,
    label: '',
    ariaLabel: 'Cart',
    role: 'button',
  },
  {
    icon: SfIconFavorite,
    label: '',
    ariaLabel: 'Wishlist',
    role: 'button',
  },
  {
    icon: SfIconPerson,
    label: 'Log in',
    ariaLabel: 'Log in',
    role: 'login',
  },
];

const { data } = await useAsyncData(async () => await sdk.magento.categoryList({}))

type MenuCategoryChild = {
  title: string
  link: string
}

const res = computed(() => data?.value?.data?.categories?.items?.[0]?.children)
const categories = res?.value?.filter(category => category?.include_in_menu == 1 && category.children?.length)
const categoriesContent = categories?.map(category => {
  return {
    heading: category?.name,
    items: getChildren(category)
  }
})

function getChildren(category) {
  const children: MenuCategoryChild[] = []
  category?.children?.map(child =>
    children.push({
      title: child?.name,
      link: `/${child.url_path}${child.url_suffix}`
    })
  )

  return children
}
</script>
