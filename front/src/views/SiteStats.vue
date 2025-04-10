<template>
  <div class="container">
    <div class="content">
      <br>
      <Row>
        <Col :xs="{push: 1}" :lg="{push: 0}">
          <Breadcrumb>
            <BreadcrumbItem :to="{name: 'home'}">{{ $t("header.index") }}</BreadcrumbItem>
            <BreadcrumbItem>{{ $t("sitestats.title") }}</BreadcrumbItem>
          </Breadcrumb>
        </Col>
      </Row>
      <br>

      <Col>
        <Card :padding="0" dis-hover>
          <Row :gutter="10" class="sitestats-chart-top">
            <Col>
              <Button @click="loadData">
                <Icon type="md-refresh" :class="[load ? 'spin-icon-load' :'']"/>
              </Button>
            </Col>
            <Col>
              <Select v-model="chartConfig.timeFrameValue" @on-change="changeChartTime">
                <Option v-for="(i,index) in chartConfig.timeFrameList" :key="index" :value="i.label">
                  <template v-if="i.label === 'all'">All</template>
                  <template v-else>
                    {{ $t(`sitestats.timeRange.${i.label}`) }}
                  </template>
                </Option>
              </Select>
            </Col>
          </Row>
          <v-chart ref="chart" class="chart" :option="chart.stats"/>
        </Card>
        <Spin size="large" fix v-if="load"></Spin>
      </Col>
      <br>

      <Card dis-hover>
        <Row type="flex" justify="space-between" align="middle">
          <Col :xs="{span: 12}" :lg="{span: 3}">
            <h2>
              {{ statistics.reports || 0 }}
            </h2>
            <span>{{ $t('sitestats.reports') }}</span>
          </Col>
          <Divider type="vertical" class="mobile-hide"/>
          <Col :xs="{span: 12}" :lg="{span: 3}">
            <h2>
              <Badge :color="chart.stats.color[0]"/>
              {{ statistics.players || 0 }}
            </h2>
            {{ $t('sitestats.players') }}
          </Col>
          <Divider type="vertical" class="mobile-hide"/>
          <Col :xs="{span: 12}" :lg="{span: 3}">
            <h2>{{ statistics.banAppeals || 0 }}</h2>
            {{ $t('sitestats.banAppeals') }}
          </Col>
          <Divider type="vertical" class="mobile-hide"/>
          <Col :xs="{span: 12}" :lg="{span: 3}">
            <h2>
              <Badge :color="chart.stats.color[2]"/>
              {{ statistics.registers || 0 }}
            </h2>
            <span>{{ $t('sitestats.registers') }}</span>
          </Col>
          <Divider type="vertical" class="mobile-hide"/>
          <Col :xs="{span: 12}" :lg="{span: 3}">
            <h2>
              <Badge :color="chart.stats.color[1]"/>
              {{ statistics.confirmed || 0 }}
            </h2>
            <span>{{ $t('sitestats.confirmed') }}</span>
          </Col>
          <Divider type="vertical" class="mobile-hide"/>
          <Col :xs="{span: 12}" :lg="{span: 3}">
            <h2>{{ statistics.admins || 0 }}</h2>
            {{ $t('sitestats.admins') }}
          </Col>
        </Row>
      </Card>
      <br>

      <!-- 排行统计 S -->
      <div style="position: relative" v-if="isLogin">
        <div>
          <Row :gutter="20">
            <Col :xs="{span: 24, push: 1}" :lg="{span: 20, push: 0}">
              <RadioGroup size="small" type="button" v-voice-button v-model="timeRange"
                          @on-change="onChangePeriod">
                <Radio v-for="(i, index) in timeArray" :key="index"
                       :disabled="!isLogin && !show"
                       :label="i.value"
                       :value="i.value">{{ $t(i.name) }}
                </Radio>
              </RadioGroup>
            </Col>
            <Col :xs="{span: 24, push: 0}" :lg="{span: 4,push: 0}" align="right" class="mobile-hide">
              <Checkbox v-model="isIncludingRobots" @on-change="onChangePeriod"></Checkbox>
              <PrivilegesTag :data="['bot']"></PrivilegesTag>
            </Col>
          </Row>
        </div>
        <br>

        <Row :gutter="20">
          <Col :xs="{span: 24}" :lg="{span: 12}">
            <Card dis-hover>
              <div slot="title">{{ $t('sitestats.communityParticipation') }} ·
                {{ $t(timeArray.find(i => i.value == timeRange).name) }}
              </div>
              <Row :gutter="5">
                <Col span="14">
                  <ol class="sitestats-ul" v-if="active.community.length > 0">
                    <li v-for="(i, index) in active.community" :key="index">
                      <Row :gutter="10" type="flex" align="middle">
                        <Col>
                          <businessCard :id="i.id">
                            <HtmlLink :text="i.username" :is-poptip="false"
                                      :href="`/space/${i.id}?username=${i.username}`"></HtmlLink>
                          </businessCard>
                        </Col>
                        <Col flex="1">
                          <Divider dashed style="margin: 0"></Divider>
                        </Col>
                        <Col>{{ Number.isInteger(i.total) ? i.total : i.total.toFixed(1) || 0 }}</Col>
                      </Row>
                    </li>
                  </ol>
                  <Empty :notHint="false" v-else></Empty>
                </Col>
                <Col span="10">
                  <v-chart class="chart chart-min" :option="active.communityConf"/>
                </Col>
              </Row>
              <Spin size="large" fix v-show="active.load">
                <Loading></Loading>
              </Spin>
            </Card>
          </Col>
          <Col :xs="{span: 24}" :lg="{span: 12}">
            <Card dis-hover>
              <div slot="title">{{ $t('sitestats.reportRanking') }} ·
                {{ $t(timeArray.find(i => i.value == timeRange).name) }}
              </div>
              <ol class="sitestats-ul" v-if="active.report.length > 0">
                <li v-for="(i, index) in active.report" :key="index">
                  <Row :gutter="10" type="flex" align="middle">
                    <Col>
                      <businessCard :id="i.id">
                        <HtmlLink :text="i.username" :is-poptip="false"
                                  :href="`/space/${i.id}?username=${i.username}`"></HtmlLink>
                      </businessCard>
                    </Col>
                    <Col flex="1">
                      <Divider dashed style="margin: 0"></Divider>
                    </Col>
                    <Col>{{ i.total.toFixed(0) || 0 }}</Col>
                  </Row>
                </li>
              </ol>
              <Empty :notHint="false" v-else></Empty>
              <Spin size="large" fix v-show="active.load">
                <Loading></Loading>
              </Spin>
            </Card>
          </Col>
        </Row>
        <br>
        <Row :gutter="20">
          <Col :xs="{span:24}" :lg="{span:12}">
            <Card dis-hover>
              <div slot="title">{{ $t('sitestats.trend') }} · {{
                  $t(timeArray.find(i => i.value == timeRange).name)
                }}
              </div>
              <TrendWidget ref="trend" :disabled="!isLogin" :timeRange="timeRange"/>
            </Card>
          </Col>
          <Col :xs="{span:24}" :lg="{span:12}">
            <Card dis-hover>
              <div slot="title">{{ $t('sitestats.achievement') }}</div>
              <ol class="sitestats-ul" v-if="active.achievement.length > 0">
                <li v-for="(i, index) in active.achievement" :key="index">
                  <Row :gutter="10" type="flex" align="middle">
                    <Col>
                      <ExposedName>
                        <HtmlLink :text="i.username" :is-poptip="false"
                                  :href="`/space/${i.id}?username=${i.username}`"></HtmlLink>
                      </ExposedName>
                    </Col>
                    <Col flex="1">
                      <Divider dashed style="margin: 0"></Divider>
                    </Col>
                    <Col>
                      <AchievementsTag :maxOverflow="10" :data="i.userAchievements" size="15"></AchievementsTag>
                    </Col>
                    <Col>
                      <Tag color="primary">
                        <Icon type="md-color-wand"/>
                        {{ i.points || 0 }}
                      </Tag>
                    </Col>
                  </Row>
                </li>
              </ol>
              <Empty :notHint="false" v-else></Empty>
              <Spin size="large" fix v-show="active.load">
                <Loading></Loading>
              </Spin>
            </Card>
          </Col>
        </Row>
      </div>
      <!-- 排行统计 E -->

      <Card v-show="!isLogin" dis-hover>
        <Spin size="large">
          <div>
            <Icon type="md-lock" size="100"/>
          </div>
          <br>
          <Button :to="{name: 'signin'}">{{ $t("header.signin") }}</Button>
        </Spin>
      </Card>

    </div>
  </div>
</template>

<script>
import {account_storage, api, application, http, time} from "@/assets/js"

import Empty from "@/components/Empty"
import businessCard from "@/components/BusinessCard.vue";
import PrivilegesTag from "@/components/PrivilegesTag.vue";
import AchievementsTag from "@/components/AchievementsTag.vue";
import HtmlLink from "@/components/HtmlLink.vue"
import TrendWidget from "@/components/TrendWidget.vue";
import ExposedName from "@/components/ExposedName.vue"
import Loading from "@/components/Loading.vue";
import "echarts";

export default new application({
  data() {
    return {
      load: false,
      statistics: {},
      show: false,
      isIncludingRobots: true,
      chart: {
        stats: {
          color: ['#fff13c', '#401486', '#ed4014'],
          tooltip: {
            trigger: 'axis',
            axisPointer: {
              type: 'cross',
              label: {
                backgroundColor: '#f2f2f2',
                color: '#535353'
              }
            }
          },
          xAxis: {
            type: 'time',
            boundaryGap: true,
            splitLine: {
              show: true,
            },
            min: () => {
              switch (this.chartConfig.timeFrameValue) {
                case 'yearly':
                  return new Date(new Date().getTime() - 360 * 24 * 3600 * 1000)
                case 'monthly':
                  return new Date(new Date().getTime() - 24 * 3600 * 1000)
                case 'all':
                default:
                  return time.appStart
              }
            },
          },
          yAxis: {
            type: 'value',
            boundaryGap: [0, '30%'],
            show: false,
          },
          series: [],
          grid: {x: -1, y: -1, x2: -1, y2: 30},
          useUTC: true
        },
      },
      chartConfig: {
        timeFrameValue: 'yearly',
        timeFrameList: [
          {
            label: 'all',
            time: 1514764800000
          },
          {
            label: 'yearly',
            time: new Date().getTime() - 360 * 24 * 3600 * 1000
          },
          {
            label: 'monthly',
            time: new Date().getTime() - 24 * 3600 * 1000
          }
        ],
        array: [{
          name: "players",
          valName: 'playerStats',
          lineColor: '#fff13c'
        }, {
          name: "confirmed",
          valName: 'confirmStats',
          lineColor: '#401486'
        }, {
          name: "registers",
          valName: 'userStats',
          lineColor: '#ed4014'
        }],
      },

      timeArray: [
        {name: 'sitestats.timeRange.daily', value: 'daily', show: false},
        {name: 'sitestats.timeRange.weekly', value: 'weekly', show: false},
        {
          name: 'sitestats.timeRange.monthly',
          value: 'monthly',
          show: false,
          privileges: ['admin', 'spuer', 'root', 'dev']
        },
        {
          name: 'sitestats.timeRange.yearly',
          value: 'yearly',
          show: false,
          privileges: ['admin', 'spuer', 'root', 'dev']
        }
      ],
      timeRange: 'weekly',

      active: {
        load: false,
        communityConf: {
          tooltip: {
            trigger: 'item'
          },
          series: [
            {
              type: 'pie',
              itemStyle: {
                borderRadius: 5,
                borderWidth: 3,
              },
              radius: ['50%', '70%'],
              avoidLabelOverlap: false,
              label: {show: false, position: 'center'},
              labelLine: {show: false},
              data: []
            }
          ]
        },
        community: Array.from({length: 10}, (_, index) => {
          return {'total': 0, 'username': `🦈 ${index}`, 'id': -1}
        }) || [],
        report: Array.from({length: 10}, (_, index) => {
          return {'total': 0, 'username': `🦖 ${index}`, 'id': -2}
        }) || [],
        achievement: Array.from({length: 10}, (_, index) => {
          return {'total': 0, 'username': `🐧 ${index}`, 'id': -3}
        }) || [],
      }
    }
  },
  created() {
    this.loadData();
  },
  components: {businessCard, Loading, Empty, PrivilegesTag, AchievementsTag, HtmlLink, TrendWidget, ExposedName},
  methods: {
    async loadData() {
      this.init();

      this.load = true;

      await this.getSiteStats();
      await this.getStatisticsInfo();
      await this.onChangePeriod();

      setTimeout(() => {
        this.load = false;
      }, 1000);
    },
    init() {
      this.chart.stats.series = [];

      this.chartConfig.array.map(i => {
        this.chart.stats.series.push({
          name: this.$i18n.t(`sitestats.${i.name}`),
          valName: i.valName,
          type: 'line',
          smooth: true,
          symbol: 'none',
          stack: 'Total',
          stackStrategy: 'positive',
          showSymbol: true,
          label: {
            show: true,
            position: "top"
          },
          lineStyle: {
            color: i.lineColor,
            width: 2
          },
          markLine: {
            symbol: ['none', 'none'],
            label: {show: true},
            data: [{xAxis: 1}, {xAxis: 3}, {xAxis: 5}, {xAxis: 7}]
          },
          areaStyle: {
            opacity: .5,
            color: i.lineColor
          },
          data: []
        });
      });

      this.timeArray.map(i => {
        i.show = account_storage.checkPrivilegeGroup(this.currentUser.userinfo, i.privileges);
      })
    },
    async onChangePeriod() {
      await this.getActiveStatistical();
      if (this.$refs.trend)
        await this.$refs.trend.getTrend();
    },
    /**
     * 获取活跃统计
     */
    async getActiveStatistical() {
      try {
        if (
            !this.isLogin &&
            !account_storage.checkPrivilegeGroup(this.currentUser.userinfo, this.timeRange.privileges)
        ) {
          // 例子
          let exp = [
            {username: 'Tom', total: 10},
            {username: 'Rudi', total: 5},
            {username: 'まどか', total: 3},
            {username: 'David', total: 5.1},
            {username: '张小明', total: 2.4},
            {username: 'Auston', total: 1},
            {username: 'Marcia', total: 1}
          ];
          this.active.report = exp;
          this.active.community = exp;
          this.active.community.forEach(i => {
            this.active.communityConf.series[0].data.push({value: i.total.toFixed(2), name: i.username})
          })
          return;
        }

        this.active.load = true;

        let selectTime = this.timeArray.filter(i => i.value === this.timeRange)[0].value;

        const result = await http.get(api['activeStatistical'], {
          params: {
            isBot: this.isIncludingRobots,
            time: selectTime,
            report: true,
            community: true,
          }
        });

        const d = result.data;

        if (d.success === 1) {
          this.active = Object.assign(this.active, d.data);

          this.active.communityConf.series[0].data = [];
          d.data.community.forEach(i => {
            this.active.communityConf.series[0].data.push({value: i.total.toFixed(2), name: i.username})
          })
        }
      } finally {
        this.active.load = false;
      }
    },
    /**
     * 更变统计视图时间范围
     */
    changeChartTime() {
      this.$refs.chart.setOption(this.chart.stats);
    },
    /**
     * 获取网站统计信息
     * api/siteStats
     */
    async getSiteStats() {
      try {
        const result = await http.get(api['siteStats']),
            d = result.data;

        if (d.error === 1)
          throw new Error(result.message);

        for (const dataKey in d.data) {
          this.chart.stats.series.forEach(i => {
            if (i.valName === dataKey)
              d.data[dataKey].forEach(t => {
                i.data.push([t.time, t.num]);
              })
          })
        }
      } catch (e) {
        if (e instanceof Error) {
          this.$Message.error(e);
          return;
        }

        this.$Message.error(e.toString());
      }
    },
    /**
     * 获取状态总数统计
     */
    async getStatisticsInfo() {
      const result = await http.get(api["statistics"], {
        params: {
          reports: true,
          players: true,
          confirmed: true,
          registers: true,
          banAppeals: true,
          admins: true,
          from: 1514764800000
        }
      })

      const d = result.data;
      if (d.success === 1) {
        this.statistics = d.data;
      }
    },
  },
  computed: {
    time: () => time
  }
});
</script>

<style lang="less" scoped>
@import "@/assets/css/icon.less";

.chart {
  height: 300px;
}

.chart-min {
  width: 100%;
  height: 350px;
}

.sitestats-ul {
  padding-left: 25px;

  li {
    padding-left: 5px;
    line-height: 2.1rem;
  }

  li:nth-child(1)::marker,
  li:nth-child(2)::marker,
  li:nth-child(3)::marker {
    font-size: 20px;
  }

  li:nth-child(3) {
    margin-bottom: 5px;
  }

  li::marker {
    font-size: 16px;
    display: block;
    padding-left: 20px;
  }
}

.sitestats-chart-top {
  position: absolute;
  z-index: 10;
  top: 10px;
  left: 10px;
}

.sitestats-avatar {
  position: relative;
  padding-top: 30px;
  padding-left: 40px;
}

.sitestats-avatar span {
  margin-left: -15px;
  margin-bottom: 10px;
  box-shadow: 0 0 10px #00000017;
}
</style>
