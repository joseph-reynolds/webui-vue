<template>
  <overview-card
    :data="dumps"
    :file-name="exportFileNameByDate()"
    :title="$t('pageOverview.dumps')"
    :to="`/logs/dumps`"
  >
    <b-row class="mt-3">
      <b-col sm="6">
        <dl>
          <dt>{{ $t('pageOverview.total') }}</dt>
          <dd class="h3">{{ dataFormatter(dumps.length) }}</dd>
        </dl>
      </b-col>
    </b-row>
  </overview-card>
</template>

<script>
import OverviewCard from './OverviewCard';
import DataFormatterMixin from '@/components/Mixins/DataFormatterMixin';

export default {
  name: 'Dumps',
  components: {
    OverviewCard,
  },
  mixins: [DataFormatterMixin],
  computed: {
    dumps() {
      return this.$store.getters['dumps/allDumps'];
    },
  },
  created() {
    this.$store.dispatch('dumps/getAllDumps').finally(() => {
      this.$root.$emit('overview-dumps-complete');
    });
  },
  methods: {
    exportFileNameByDate() {
      // Create export file name based on date
      let date = new Date();
      date =
        date.toISOString().slice(0, 10) +
        '_' +
        date.toString().split(':').join('-').split(' ')[4];
      let fileName = 'all_dumps_';
      return fileName + date;
    },
  },
};
</script>
