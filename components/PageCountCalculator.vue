<script setup lang="ts">
  // general
  const { t } = useI18n();
  const months = [
    'january',
    'february',
    'march',
    'april',
    'may',
    'june',
    'july',
    'august',
    'september',
    'october',
    'november',
    'december',
  ];

  // tabs
  const items = [
    { label: t('pageCountCalculator.tabs.form.label'), key: 'form' },
    { label: t('pageCountCalculator.tabs.importExport.label'), key: 'importExport' },
  ];
  const selectedTab = ref<number>(0);
  const selectedItem = computed(() => items[selectedTab.value]);

  // form
  type YearlySection = {
    name: string;
    pages: number;
  };
  type MonthlySection = {
    name: string;
    pages: number;
  };
  const form = ref<{
    jounalPageAmount;
    year: number;
    startMonth: string;
    firstPagePosition: 'left' | 'right';
    addLegendPage: boolean;
    addMonthlyCalendar: boolean;
    excludeMonthsForMonthlyCalendar: string[];
    monthlyCalendarLayout?: '8gridPerWeek';
    yearlySections: YearlySection[];
    monthlySections: MonthlySection[];
  }>({
    jounalPageAmount: 203,
    year: new Date().getFullYear(),
    startMonth: 'february',
    firstPagePosition: 'left',
    addLegendPage: true,
    addMonthlyCalendar: true,
    excludeMonthsForMonthlyCalendar: ['february'],
    monthlyCalendarLayout: '8gridPerWeek',
    yearlySections: [],
    monthlySections: [],
  });
  const yearOptions = Array.from({ length: 20 }, (_, index) => ({
    value: new Date().getFullYear() + index - 10,
    label: (new Date().getFullYear() + index - 10).toString(),
  }));
  const startMonthOptions = months.map((month) => ({
    value: month,
    label: t(`month.${month}.long`),
  }));
  const firstPagePositionOptions = [
    { value: 'left', label: t('pageCountCalculator.form.firstPagePosition.option.left.label') },
    { value: 'right', label: t('pageCountCalculator.form.firstPagePosition.option.right.label') },
  ];
  const monthlyCalendarLayoutOptions = [
    {
      value: '8gridPerWeek',
      label: t('pageCountCalculator.form.monthlyCalendarLayout.option.8gridPerWeek.label'),
    },
  ];
  const yearlySectionsAmount = ref<number>(0);
  const addYearlySection = ({ name = '', pages = 2 } = {}) => {
    form.value.yearlySections.push({ name, pages });
    yearlySectionsAmount.value++;
  };
  const removeYearlySection = ({ index }: { index: number }) => {
    yearlySectionsAmount.value--;
    form.value.yearlySections.splice(index, 1);
  };
  const monthlySectionsAmount = ref<number>(0);
  const addMonthlySection = ({ name = '', pages = 2 } = {}) => {
    form.value.monthlySections.push({ name, pages });
    monthlySectionsAmount.value++;
  };
  const removeMonthlySection = ({ index }: { index: number }) => {
    monthlySectionsAmount.value--;
    form.value.monthlySections.splice(index, 1);
  };
  addYearlySection({
    name: 'Bucket-Liste',
    pages: 2,
  });
  addYearlySection({
    name: 'To-Do-Liste',
    pages: 2,
  });
  addYearlySection({
    name: 'Bücher',
    pages: 2,
  });
  addMonthlySection({
    name: 'Monatsübersicht',
    pages: 1,
  });
  addMonthlySection({
    name: 'Ziele',
    pages: 1,
  });
  addMonthlySection({
    name: 'Habit Tracker',
    pages: 2,
  });

  // result
  const monthsForResult = computed<string[]>(() => {
    const startMonthIndex = months.indexOf(form.value.startMonth);
    return months.slice(startMonthIndex);
  });
  const formResult = computed<{
    pages: {
      dividerBefore?: boolean;
      start: number;
      end: number;
      title: string;
    }[];
  }>(() => {
    const pages: {
      dividerBefore?: boolean;
      start: number;
      end: number;
      title: string;
    }[] = [];

    // init
    let page = 1;
    let dividerBefore = false;

    // first page
    if (form.value.addLegendPage) {
      pages.push({ start: page, end: page, title: 'Legende', dividerBefore });
      page++;
    }

    // yearly sections
    dividerBefore = true;
    for (const section of form.value.yearlySections) {
      if (!section.pages) {
        continue;
      }
      pages.push({
        start: page,
        end: page + section.pages - 1,
        title: section.name,
        dividerBefore,
      });
      page += section.pages;
      dividerBefore = false;
    }

    // monthly sections
    for (const month of monthsForResult.value) {
      dividerBefore = true;
      for (const section of form.value.monthlySections) {
        if (!section.pages) {
          continue;
        }
        pages.push({
          start: page,
          end: page + section.pages - 1,
          title: `${t(`month.${month}.long`)} - ${section.name}`,
          dividerBefore,
        });
        page += section.pages;
        dividerBefore = false;
      }

      const getWeekNumber = (date: Date) => {
        const d = new Date(Date.UTC(date.getFullYear(), date.getMonth(), date.getDate()));
        d.setUTCDate(d.getUTCDate() + 4 - (d.getUTCDay() || 7));
        const yearStart = new Date(Date.UTC(d.getUTCFullYear(), 0, 1));
        return Math.ceil(((d.getTime() - yearStart.getTime()) / 86400000 + 1) / 7);
      };

      if (
        form.value.addMonthlyCalendar &&
        !form.value.excludeMonthsForMonthlyCalendar.includes(month)
      ) {
        // get first and last day of month
        const firstDay = new Date(form.value.year, months.indexOf(month), 1);

        const lastDay = new Date(form.value.year, months.indexOf(month) + 1, 0);

        // get first and last calendar week of month
        const firstWeek = getWeekNumber(firstDay);
        const lastWeek = getWeekNumber(lastDay);
        const yearChange = lastWeek < firstWeek;

        // get highest week number of year
        let lastWeekOfYear = 0;
        for (let i = 31; i > 0; i--) {
          const d = new Date(form.value.year, 11, i);
          if (getWeekNumber(d) > lastWeekOfYear) {
            lastWeekOfYear = getWeekNumber(d);
          }
        }

        // check layout
        if (form.value.monthlyCalendarLayout === '8gridPerWeek') {
          // 8 grid per week
          for (let week = firstWeek; week <= (yearChange ? lastWeekOfYear : lastWeek); week++) {
            pages.push({
              start: page,
              end: page + 1,
              title: `${t(`month.${month}.long`)} - ${t(
                'pageCountCalculator.form.result.monthlyCalendar.week.title'
              )} ${week}`,
            });
            page += 2;
          }

          if (yearChange) {
            for (let week = 1; week <= lastWeek; week++) {
              pages.push({
                start: page,
                end: page + 1,
                title: `${t(`month.${month}.long`)} - ${t(
                  'pageCountCalculator.form.result.monthlyCalendar.week.title'
                )} ${week}`,
              });
              page += 2;
            }
          }
        }
      }
    }

    return { pages };
  });

  // json value
  const jsonValue = ref<string>('');
  const lockUpdateJsonValue = ref<boolean>(false);
  const updateJsonValue = () => {
    if (lockUpdateJsonValue.value) {
      return;
    }
    jsonValue.value = JSON.stringify(form.value, null, 2);
  };
  updateJsonValue();
  watch(
    form,
    () => {
      updateJsonValue();
    },
    { deep: true }
  );
  watch(jsonValue, () => {
    lockUpdateJsonValue.value = true;
    try {
      const parsedValue = JSON.parse(jsonValue.value);
      form.value = parsedValue;
      yearlySectionsAmount.value = parsedValue.yearlySections.length;
    } catch (error) {
      console.error(error);
    }
    lockUpdateJsonValue.value = false;
  });

  // footer links
  const { locale } = useI18n();
  const imprintLink = computed(() => {
    if (locale.value == 'de-DE') {
      return 'https://kirch.dev/de/imprint';
    }

    return 'https://kirch.dev/en/imprint';
  });
</script>

<template>
  <UContainer>
    <UCard>
      <template #header>
        {{ $t('pageCountCalculator.header.title') }}
      </template>

      <UTabs v-model="selectedTab" :items="items" class="w-full mb-8" />

      <UContainer v-if="selectedItem.key === 'form'">
        <h2 class="text-xl font-semibold mb-6">
          {{ $t('pageCountCalculator.form.title') }}
        </h2>
        <UForm :state="form" class="space-y-8">
          <h3 class="text-lg font-semibold">
            {{ $t('pageCountCalculator.form.general.title') }}
          </h3>
          <UFormGroup
            name="jounalPageAmount"
            :label="$t('pageCountCalculator.form.jounalPageAmount.label')"
          >
            <UInput v-model="form.jounalPageAmount" type="number" />
          </UFormGroup>

          <UFormGroup name="year" :label="$t('pageCountCalculator.form.year.label')">
            <USelectMenu
              v-model="form.year"
              :options="yearOptions"
              value-attribute="value"
              searchable
            />
          </UFormGroup>

          <UFormGroup name="startMonth" :label="$t('pageCountCalculator.form.startMonth.label')">
            <USelectMenu
              v-model="form.startMonth"
              :options="startMonthOptions"
              value-attribute="value"
              searchable
            />
          </UFormGroup>

          <UFormGroup
            name="firstPagePosition"
            :label="$t('pageCountCalculator.form.firstPagePosition.label')"
          >
            <USelectMenu
              v-model="form.firstPagePosition"
              :options="firstPagePositionOptions"
              value-attribute="value"
              :disabled="true"
            />
          </UFormGroup>

          <UFormGroup
            name="addLegendPage"
            :label="$t('pageCountCalculator.form.addLegendPage.label')"
          >
            <UToggle v-model="form.addLegendPage" />
          </UFormGroup>

          <UFormGroup
            name="addMonthlyCalendar"
            :label="$t('pageCountCalculator.form.addMonthlyCalendar.label')"
          >
            <UToggle v-model="form.addMonthlyCalendar" />
          </UFormGroup>

          <UFormGroup
            v-if="form.addMonthlyCalendar"
            name="monthlyCalendarLayout"
            :label="$t('pageCountCalculator.form.monthlyCalendarLayout.label')"
          >
            <USelectMenu
              v-model="form.monthlyCalendarLayout"
              :options="monthlyCalendarLayoutOptions"
              value-attribute="value"
              searchable
            />
          </UFormGroup>

          <UFormGroup
            v-if="form.addMonthlyCalendar"
            name="excludeMonthsForMonthlyCalendar"
            :label="$t('pageCountCalculator.form.excludeMonthsForMonthlyCalendar.label')"
          >
            <USelectMenu
              v-model="form.excludeMonthsForMonthlyCalendar"
              :options="startMonthOptions"
              value-attribute="value"
              multiple
              searchable
            />
          </UFormGroup>

          <div class="space-y-4">
            <h3 class="text-lg font-semibold">
              {{ $t('pageCountCalculator.form.yearlySections.title') }}
            </h3>
            <div v-for="index in yearlySectionsAmount" class="grid md:grid-cols-9 gap-4">
              <UFormGroup
                :name="'yearlySections[' + (index - 1) + '].name'"
                :label="$t('pageCountCalculator.form.yearlySections.name.label')"
                class="col-span-4"
              >
                <UInput v-model="form.yearlySections[index - 1].name" />
              </UFormGroup>
              <UFormGroup
                :name="'yearlySections[' + (index - 1) + '].pages'"
                :label="$t('pageCountCalculator.form.yearlySections.pages.label')"
                class="col-span-4"
              >
                <UInput v-model="form.yearlySections[index - 1].pages" type="number" />
              </UFormGroup>
              <div class="col-span-1 flex items-end justify-end">
                <UButton
                  @click="removeYearlySection({ index: index - 1 })"
                  color="red"
                  icon="i-heroicons-trash"
                />
              </div>
            </div>

            <UButton @click="addYearlySection">
              {{ $t('pageCountCalculator.form.yearlySections.add.label') }}
            </UButton>
          </div>

          <div class="space-y-4">
            <h3 class="text-lg font-semibold">
              {{ $t('pageCountCalculator.form.monthlySections.title') }}
            </h3>
            <div v-for="index in monthlySectionsAmount" class="grid md:grid-cols-9 gap-4">
              <UFormGroup
                :name="'monthlySections[' + (index - 1) + '].name'"
                :label="$t('pageCountCalculator.form.monthlySections.name.label')"
                class="col-span-4"
              >
                <UInput v-model="form.monthlySections[index - 1].name" />
              </UFormGroup>
              <UFormGroup
                :name="'monthlySections[' + (index - 1) + '].pages'"
                :label="$t('pageCountCalculator.form.monthlySections.pages.label')"
                class="col-span-4"
              >
                <UInput v-model="form.monthlySections[index - 1].pages" type="number" />
              </UFormGroup>
              <div class="col-span-1 flex items-end justify-end">
                <UButton
                  @click="removeMonthlySection({ index: index - 1 })"
                  color="red"
                  icon="i-heroicons-trash"
                />
              </div>
            </div>

            <UButton @click="addMonthlySection">
              {{ $t('pageCountCalculator.form.monthlySections.add.label') }}
            </UButton>
          </div>
        </UForm>

        <h2 class="text-xl font-semibold mb-6 mt-12">
          {{ $t('pageCountCalculator.form.result.title') }}
        </h2>
        <UContainer class="space-y-10">
          <UAlert
            :color="
              formResult.pages[formResult.pages.length - 1]?.end > form.jounalPageAmount
                ? 'red'
                : 'green'
            "
            :icon="
              formResult.pages[formResult.pages.length - 1]?.end > form.jounalPageAmount
                ? 'i-heroicons-exclamation-circle'
                : 'i-heroicons-check'
            "
            :title="
              formResult.pages[formResult.pages.length - 1]?.end > form.jounalPageAmount
                ? $t('pageCountCalculator.form.result.alert.maxPages.title', {
                    pages: formResult.pages[formResult.pages.length - 1]?.end,
                    maxPages: form.jounalPageAmount,
                    missingPages:
                      formResult.pages[formResult.pages.length - 1]?.end - form.jounalPageAmount,
                  })
                : $t('pageCountCalculator.form.result.alert.enoughPages.title', {
                    pages: formResult.pages[formResult.pages.length - 1]?.end,
                    maxPages: form.jounalPageAmount,
                    pagesLeft:
                      form.jounalPageAmount - formResult.pages[formResult.pages.length - 1]?.end,
                  })
            "
          />

          <div class="grid md:grid-cols-3 gap-4">
            <div class="col-span-1 font-semibold">
              {{ $t('pageCountCalculator.form.result.pages.title') }}
            </div>
            <div class="col-span-2 font-semibold">
              {{ $t('pageCountCalculator.form.result.content.title') }}
            </div>
          </div>

          <div v-for="page in formResult.pages" class="grid md:grid-cols-3 gap-4">
            <div v-if="page.dividerBefore" class="col-span-3 border-b-2 border-gray-200 mt-4" />
            <div class="col-span-1">
              <span class="font-semibold">{{ page.start }}</span>
              <template v-if="page.start !== page.end">
                <span>-</span>
                <span class="font-semibold">{{ page.end }}</span>
              </template>
            </div>
            <div class="col-span-2">{{ page.title }}</div>
          </div>
        </UContainer>
      </UContainer>
      <UContainer v-else-if="selectedItem.key === 'importExport'">
        <UFormGroup
          name="importExport"
          :label="$t('pageCountCalculator.importExport.textarea.label')"
        >
          <UTextarea v-model="jsonValue" :rows="20" />
        </UFormGroup>
      </UContainer>

      <template #footer>
        <UButton :to="imprintLink" variant="link">
          {{ $t('pageCountCalculator.footer.imprint.label') }}
        </UButton>
      </template>
    </UCard>
  </UContainer>
</template>
