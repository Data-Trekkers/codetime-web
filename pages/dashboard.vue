<template>
  <v-row>
    <v-scroll-x-transition>
      <v-alert
        v-if="empty"
        style="
          position: fixed;
          z-index: 2;
          margin-left: 24px;
          top: 64px;
          right: 24px;
        "
        prominent
        dense
        type="info"
        transition="scale-transition"
      >
        <v-row align="center">
          <v-col class="grow">
            {{ $t("not found data") }}
          </v-col>
          <v-col class="shrink">
            <v-btn to="/account" text> {{ $t("get token") }} </v-btn>
          </v-col>
        </v-row>
      </v-alert>
    </v-scroll-x-transition>
    <v-col cols="12" md="4">
      <v-card outlined>
        <v-card-title>{{ $t("language rate") }}</v-card-title>
        <chart-card-content :options="langRadioOptions"></chart-card-content>
      </v-card>
    </v-col>
    <v-col cols="12" md="8">
      <v-card outlined>
        <v-card-title>{{ $t("lang usage") }}</v-card-title>
        <chart-card-content :options="languageOptions"> </chart-card-content>
      </v-card>
    </v-col>
    <v-col cols="12" md="12">
      <v-card outlined>
        <v-card-title>{{ $t("code time by day") }}</v-card-title>
        <chart-card-content :options="codeTimeDayOption"> </chart-card-content>
      </v-card>
    </v-col>
    <v-col cols="12">
      <v-card outlined>
        <v-card-title>{{ $t("code time by hour") }}</v-card-title>
        <chart-card-content :options="codeTimeHourOption"> </chart-card-content>
      </v-card>
    </v-col>

    <v-col cols="12" md="6">
      <v-card outlined>
        <v-card-title>{{ $t("editor usage") }}</v-card-title>
        <chart-card-content :options="editorOptions"> </chart-card-content>
      </v-card>
    </v-col>
    <v-col cols="12" md="6">
      <v-card outlined>
        <v-card-title>{{ $t("platform usage") }}</v-card-title>
        <chart-card-content :options="platformOptions"> </chart-card-content>
      </v-card>
    </v-col>
    <v-col cols="12" md="12">
      <v-card outlined>
        <v-card-title>{{ $t("code calendar") }}</v-card-title>
        <chart-card-content
          id="calendar"
          :height="calendarHeight"
          :options="codeTimeDayCalendarOption"
        ></chart-card-content>
      </v-card>
    </v-col>
  </v-row>
</template>

<script lang="ts">
import Vue from "vue";
import { EChartsOption } from "echarts";
import ChartCardContent from "@/components/ChartCardContent.vue";
import { getPieOptionsFromData } from "@/middleware/charts/getPieOptionsFromData";
import { getCodeTimeOptions } from "@/middleware/charts/getCodeTimeOptions";
import { getCalendarOptions } from "@/middleware/charts/getCalendarOptions";
import { getGMTTimeZone } from "@/middleware/utils/getGMTTimeZone";
import { getStackOptions } from "@/middleware/charts/getStackOptions";
export default Vue.extend({
  components: { ChartCardContent },
  data() {
    return {
      langRadioOptions: {} as EChartsOption,
      codeTimeDayOption: {} as EChartsOption,
      codeTimeHourOption: {} as EChartsOption,
      platformOptions: {} as EChartsOption,
      codeTimeDayCalendarOption: {} as EChartsOption,
      editorOptions: {} as EChartsOption,
      languageOptions: {} as EChartsOption,
      calendarHeight: 250,
      empty: false,
    };
  },
  mounted() {
    this.$axios
      .$get(`/stats/language?tz=${encodeURIComponent(getGMTTimeZone())}`)
      .then((d) => {
        if (d.data.length === 0) {
          this.empty = true;
        }
        this.langRadioOptions = getPieOptionsFromData(
          d.data,
          "language",
          "duration"
        );
      });
    this.$axios
      .$post(`/stats/byTime`, {
        timeFormat: "%Y-%m-%d %H",
        tz: getGMTTimeZone(),
      })
      .then((d) => {
        d.data.forEach((d: any) => {
          if (d.duration > 3600 * 1000) {
            d.duration = 3600 * 1000;
          }
        });
        this.codeTimeHourOption = getCodeTimeOptions(d.data);
      });
    this.$axios
      .$post(`/stats/byTime`, {
        timeFormat: "%Y-%m-%d",
        tz: getGMTTimeZone(),
      })
      .then((d) => {
        this.codeTimeDayOption = getCodeTimeOptions(d.data, "line");
        const doc = document.querySelector("#calendar") as HTMLDivElement;
        const width = doc.offsetWidth - 32;
        this.calendarHeight = (width / 53) * 7 + 20;
        this.codeTimeDayCalendarOption = getCalendarOptions(d.data, width);
      });
    this.$axios
      .$get(`/stats/editor?byDay=1&tz=${encodeURIComponent(getGMTTimeZone())}`)
      .then((d) => {
        this.editorOptions = getStackOptions(d, "editor");
      });
    this.$axios
      .$get(
        `/stats/platform?byDay=1&tz=${encodeURIComponent(getGMTTimeZone())}`
      )
      .then((d) => {
        this.platformOptions = getStackOptions(d, "platform");
      });
    this.$axios
      .$get(
        `/stats/language?byDay=1&tz=${encodeURIComponent(getGMTTimeZone())}`
      )
      .then((d) => {
        this.languageOptions = getStackOptions(d, "language");
      });
  },
});
</script>
