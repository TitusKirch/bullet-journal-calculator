<script setup lang="ts">
  const route = useRoute();
  const { t } = useI18n();
  const head = useLocaleHead({
    addDirAttribute: true,
    identifierAttribute: 'id',
    addSeoAttributes: true,
  });

  const title = computed(() => {
    if (typeof route?.meta?.title === 'string') {
      return t('app.meta.title', { title: t(route?.meta?.title ?? 'UNKNOWN') });
    }

    return t('app.meta.fallbackTitle');
  });
</script>

<template>
  <div>
    <Html :lang="head.htmlAttrs.lang" :dir="head.htmlAttrs.dir">
      <Head>
        <Title>{{ title }}</Title>
        <template v-for="link in head.link" :key="link.id">
          <Link :id="link.id" :rel="link.rel" :href="link.href" :hreflang="link.hreflang" />
        </template>
        <template v-for="meta in head.meta" :key="meta.id">
          <Meta :id="meta.id" :property="meta.property" :content="meta.content" />
        </template>
      </Head>
      <Body class="dark:bg-gray-950 pt-16">
        <NuxtLayout>
          <NuxtPage />
        </NuxtLayout>
      </Body>
    </Html>
  </div>
</template>
