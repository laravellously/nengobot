<template lang='jade'>
  .contain.py2
    .text(v-html='text')
    .hr
    h3 Market watchers
    .text(v-if='!watchers.length')
      p You are currently not watching any markets.
    table.full.clickable(v-if='watchers.length')
      thead
        tr
          th exchange
          th currency
          th asset
          th started at
          th last update
          th duration
      tbody
        tr.clickable(v-for='gekko in watchers', v-on:click='$router.push({path: `live-gekkos/watcher/${gekko.id}`})')
          td {{ gekko.watch.exchange }}
          td {{ gekko.watch.currency }}
          td {{ gekko.watch.asset }}
          td 
            template(v-if='gekko.firstCandle') {{ fmt(gekko.firstCandle.start) }}
          td
            template(v-if='gekko.lastCandle') {{ fmt(gekko.lastCandle.start) }}
          td
            template(v-if='gekko.firstCandle && gekko.lastCandle') {{ timespan(gekko.lastCandle.start, gekko.firstCandle.start) }}
    h3 Strat runners
    .text(v-if='!stratrunners.length')
      p You are currently not running any strategies.
    table.full(v-if='stratrunners.length')
      thead
        tr
          th id
          th exchange
          th currency
          th asset
          th last update
          th duration
          th strategy
          th profit
          th delete?
      tbody
        tr
          td.clickable(v-for='gekko in stratrunners', v-on:click='$router.push({path: `live-gekkos/stratrunner/${gekko.id}`})') {{ gekko.id }}
          td {{ gekko.watch.exchange }}
          td {{ gekko.watch.currency }}
          td {{ gekko.watch.asset }}
          td
            template(v-if='gekko.lastCandle') {{ fmt(gekko.lastCandle.start) }}
          td
            template(v-if='gekko.firstCandle && gekko.lastCandle') {{ timespan(gekko.lastCandle.start, gekko.firstCandle.start) }}
          td {{ gekko.strat.name }}
          td
            template(v-if='!gekko.report') 0
            template(v-if='gekko.report') {{ round(gekko.report.profit) }} {{ gekko.watch.currency }}
          td
            router-link.btn--danger(to='api/killGekko/${gekko.id}') Kill This Gekko
    .hr
    h2 Start a new live Gekko
    router-link.btn--primary(to='/live-gekkos/new') Start a new live Gekko!
</template>

<script>

import marked from '../../tools/marked'
// global moment
// global humanizeDuration

const text = marked(`

## Live Gekko

Run your strategy against live market data!

`);

export default {
  data: () => {
    return {
      text
    }
  },
  created: function() {
    this.timer = setInterval(() => {
      this.now = moment();
    }, 1000)
  },
  destroyed: function() {
    clearTimeout(this.timer);
  },
  data: () => {
    return {
      text,
      timer: false,
      now: moment()
    }
  },
  computed: {
    stratrunners: function() {
      return this.$store.state.stratrunners
    },
    watchers: function() {
      return this.$store.state.watchers
    }
  },
  methods: {
    humanizeDuration: (n) => window.humanizeDuration(n),
    moment: mom => moment.utc(mom),
    fmt: mom => moment.utc(mom).format('YYYY-MM-DD HH:mm'),
    round: n => (+n).toFixed(3),
    timespan: function(a, b) {
      return this.humanizeDuration(this.moment(a).diff(this.moment(b)))
    }
  }
}
</script>

<style>
table.clickable {
  border-collapse: separate;
}

td.clickable td:nth-child(1) {
  padding-left: 5px;
}

td.clickable {
  cursor: pointer;
}
td.clickable:hover {
  background: rgba(216,216,216,.99);
}
</style>
