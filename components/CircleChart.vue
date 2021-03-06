<template>
  <data-view :title="title" :title-id="titleId" :date="date" :url="url">
    <template v-slot:button>
      <ul :class="$style.GraphDesc">
        <i18n tag="li" path="（注）病床数は兵庫県が公開する{data}を参照">
          <a
            :class="$style.GraphLink"
            href="https://web.pref.hyogo.lg.jp/kk03/documents/corona200324-2-1_1.pdf"
            target="_blank"
            rel="noopener"
            place="data"
          >
            {{ $t('PDF') }}
          </a>
        </i18n>
      </ul>
    </template>
    <pie-chart
      :style="{ display: canvas ? 'block' : 'none' }"
      :chart-id="chartId"
      :chart-data="displayData"
      :options="displayOption"
      :height="240"
    />
    <v-data-table
      :style="{ top: '-9999px', position: canvas ? 'fixed' : 'static' }"
      :headers="tableHeaders"
      :items="tableData"
      :items-per-page="-1"
      :hide-default-footer="true"
      :height="240"
      :fixed-header="true"
      :mobile-breakpoint="0"
      class="cardTable"
      item-key="name"
    />
    <template v-slot:infoPanel>
      <data-view-basic-info-panel
        :l-text="displayInfo.lText"
        :s-text="displayInfo.sText"
        :unit="displayInfo.unit"
      />
    </template>
    <template v-slot:footer>
      <open-data-link v-show="url" :url="url" />
    </template>
  </data-view>
</template>

<script lang="ts">
import Vue from 'vue'
import { ThisTypedComponentOptionsWithRecordProps } from 'vue/types/options'
import { GraphDataType } from '@/utils/formatVariableGraph'
import DataView from '@/components/DataView.vue'
import DataViewBasicInfoPanel from '@/components/DataViewBasicInfoPanel.vue'
import { doubleGray as colors } from '@/utils/colors'
import OpenDataLink from '@/components/OpenDataLink.vue'

interface HTMLElementEvent<T extends HTMLElement> extends Event {
  currentTarget: T
}
type Data = {
  dataKind: 'transition' | 'cumulative'
  canvas: boolean
}
type Methods = {}
type Computed = {
  displayInfo: {
    lText: string
    sText: string
    unit: string
  }
  displayData: {
    labels: string[]
    datasets: {
      label: string[]
      data: number[]
      backgroundColor: string[]
    }[]
  }
  displayOption: {
    tooltips: {
      displayColors: boolean
      callbacks: {
        label: (tooltipItem: any) => string
        title: (tooltipItem: any, data: any) => string
      }
    }
    responsive: boolean
    maintainAspectRatio: boolean
    legend: {
      display: boolean
      onHover: (e: HTMLElementEvent<HTMLElement>) => void
      onLeave: (e: HTMLElementEvent<HTMLElement>) => void
    }
  }
  tableHeaders: {
    text: string
    value: string
  }[]
  tableData: {
    [key: number]: number
  }[]
}

type Props = {
  title: string
  titleId: string
  chartId: string
  chartData: GraphDataType[]
  date: string
  unit: string
  info: string
  url: string
}

const options: ThisTypedComponentOptionsWithRecordProps<
  Vue,
  Data,
  Methods,
  Computed,
  Props
> = {
  created() {
    this.canvas = process.browser
  },
  components: { DataView, DataViewBasicInfoPanel, OpenDataLink },
  props: {
    title: {
      type: String,
      default: ''
    },
    titleId: {
      type: String,
      default: ''
    },
    chartId: {
      type: String,
      default: 'pie-chart'
    },
    chartData: {
      type: Array,
      required: false,
      default: () => []
    },
    date: {
      type: String,
      required: true,
      default: ''
    },
    unit: {
      type: String,
      default: ''
    },
    info: {
      type: String,
      default: ''
    },
    url: {
      type: String,
      required: false,
      default: ''
    }
  },
  data: () => ({
    dataKind: 'transition',
    canvas: true
  }),
  computed: {
    displayInfo() {
      return {
        lText: this.chartData[
          this.chartData.length - 1
        ].cumulative.toLocaleString(),
        sText: this.info,
        unit: this.unit
      }
    },
    displayData() {
      return {
        labels: this.chartData.map(d => {
          return d.label
        }),
        datasets: [
          {
            label: this.chartData.map(d => {
              return d.label
            }),
            data: this.chartData.map(d => {
              return d.transition
            }),
            backgroundColor: this.chartData.map((d, index) => {
              return colors[index]
            }),
            borderWidth: 0
          }
        ]
      }
    },
    displayOption() {
      const unit = this.unit
      const chartData = this.chartData
      const options = {
        tooltips: {
          displayColors: false,
          callbacks: {
            label(tooltipItem: any) {
              /* return (
                parseInt(
                  chartData[tooltipItem.index].transition
                ).toLocaleString() + unit
              ) */
              return `${chartData[tooltipItem.index].transition} ${
                tooltipItem.index === 1 ? unit : '人'
              } (総病床数: ${chartData[0].transition +
                chartData[1].transition}${unit})`
            },
            title(tooltipItem: any, data: any) {
              return data.labels[tooltipItem[0].index]
            }
          }
        },
        responsive: true,
        maintainAspectRatio: false,
        legend: {
          display: true,
          onHover: (e: HTMLElementEvent<HTMLElement>): void => {
            e.currentTarget.style.cursor = 'pointer'
          },
          onLeave: (e: HTMLElementEvent<HTMLElement>): void => {
            e.currentTarget.style.cursor = 'default'
          }
        }
      }
      if (this.$route.query.ogp === 'true') {
        Object.assign(options, { animation: { duration: 0 } })
      }
      return options
    },
    tableHeaders() {
      return [
        { text: '', value: 'text' },
        ...this.chartData.map((d, index) => {
          return { text: d.label, value: String(index) }
        })
      ]
    },
    tableData() {
      return this.chartData.map((_, i) => {
        return Object.assign(
          { text: this.chartData[i].label },
          { [i]: this.chartData[i].transition }
        )
      })
    }
  }
}

export default Vue.extend(options)
</script>

<style module lang="scss">
.Graph {
  &Desc {
    width: 100%;
    margin: 0;
    margin-bottom: 0 !important;
    padding-left: 0 !important;
    font-size: 12px;
    color: $gray-3;
    list-style: none;
  }
  &Link {
    text-decoration: none;
  }
}
</style>
