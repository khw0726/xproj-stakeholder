<template>
  <v-flex d-flex xs6>
    <v-card ripple @click.capture="onStakeholderOverviewItemClick">
      <v-card-title primary>
        <span class="name">{{stakeholder.name}}</span>
      </v-card-title>
      <!-- <v-card-text>
        <v-layout row wrap>
          <v-flex v-bind="{[`xs${positiveCols}`]: true}">
            <v-card color="blue lighten-3">
              <v-card-text>
                ?
              </v-card-text>
            </v-card>
          </v-flex>
          <v-flex v-bind="{[`xs${negativeCols}`]: true}">
            <v-card color="red lighten-3">
              <v-card-text>
                ?
              </v-card-text>
            </v-card>
          </v-flex>
        </v-layout>
      </v-card-text> -->
      <!-- <v-card-text>
        <small> +: {{positiveText}}</small>
        <br>
        <small> -: {{negativeText}}</small>
      </v-card-text> -->
      <v-card-text class="centre">
        <!-- <v-slider readonly :value="negativeCols / (positiveCols + negativeCols) * 100" prepend-icon="mood_bad" append-icon="mood">
        </v-slider> -->
        <v-icon style="color:blue;">sentiment_very_satisfied</v-icon> {{poscnt}}개 &nbsp;
        <v-icon style="color:red;">sentiment_very_dissatisfied</v-icon> {{negcnt}}개
      </v-card-text>
    </v-card>
  </v-flex>
</template>
<script>
export default {
  props: {
    stakeholder: {
      validator: function (obj) {
        return ('name' in obj) && ('keywords' in obj)
      }
    }
  },
  computed: {
    positiveText: function () {
      if (this.stakeholder.keywords.positive.length > 33) {
        return this.stakeholder.keywords.positive.slice(0, 30).concat('...')
      } else {
        return this.stakeholder.keywords.positive
      }
    },
    negativeText: function () {
      if (this.stakeholder.keywords.negative.length > 33) {
        return this.stakeholder.keywords.negative.slice(0, 30) + '...'
      } else {
        return this.stakeholder.keywords.negative
      }
    },
    sentiment: function () {
      const pos = this.stakeholder.counts.positive
      const neg = this.stakeholder.counts.negative
      const ratio = neg / (neg + pos)

      if (ratio > 0.75) {
        return 'sentiment_very_dissatisfied'
      } else if (ratio > 0.5) {
        return 'sentiment_dissatisfied'
      } else if (ratio > 0.25) {
        return 'sentiment_satisfied'
      } else {
        return 'sentiment_very_satisfied'
      }
    },
    poscnt: function () {
      return this.stakeholder.counts.positive
    },
    negcnt: function () {
      return this.stakeholder.counts.negative
    }
  },
  methods: {
    onStakeholderOverviewItemClick: function () {
      this.$emit('stakeholder-item-click')
    }
  }
}
</script>
<style scoped>
.name {
  cursor: pointer;
  word-break: keep-all;
}
small {
  cursor: pointer;
}
.centre {
  text-align: center !important;
  vertical-align: center;
  cursor: pointer;
  padding-left: 10px;
  padding-right: 10px;
}
</style>

