<template>
  <div class="container">
    <div class="content">
      <template v-if="!isFull">
        <br>
        <Row>
          <Col :xs="{push: 1}" :lg="{push: 0}">
            <Breadcrumb>
              <BreadcrumbItem :to="{name: 'home'}">{{ $t("header.index") }}</BreadcrumbItem>
              <BreadcrumbItem :to="{name: 'player_list'}">{{ $t("list.title") }}</BreadcrumbItem>
              <BreadcrumbItem :to="{name: 'player', params: { ouid: $route.params.ouid }}">
                {{ $t("detail.info.cheatersInfo") }}
              </BreadcrumbItem>
              <BreadcrumbItem>{{ $t("detail.info.appeal") }}</BreadcrumbItem>
            </Breadcrumb>
          </Col>
        </Row>
        <br>
      </template>

      <Card dis-hover>
        <h2><a href="javascript:void(0)">#</a> {{ $t('detail.appeal.modal.modalTitle') }} </h2>
        <br>
        <h3> {{ $t('detail.appeal.modal.describe') }} </h3>
        <br>

        <Row :gutter="30" style="padding: 0 15px">
          <Col flex="1">
            <ul>
              <h3>
                {{ $t('detail.appeal.modal.effectiveEvidence.title') }}
                <Icon type="md-done-all" color="#19be6b"/>
              </h3>
              <ol>
                <li>{{ $t('detail.appeal.modal.effectiveEvidence.1') }}</li>
                <li>{{ $t('detail.appeal.modal.effectiveEvidence.2') }}</li>
                <li>{{ $t('detail.appeal.modal.effectiveEvidence.3') }}</li>
              </ol>
              <br>

              <h3>
                {{ $t('detail.appeal.modal.auxiliaryProof.title') }}
                <Icon type="md-done-all" color="#19be6b"/>
              </h3>
              <ol>
                <li>{{ $t('detail.appeal.modal.auxiliaryProof.1') }}</li>
                <li>{{ $t('detail.appeal.modal.auxiliaryProof.2') }}</li>
              </ol>
            </ul>
          </Col>
          <Col flex="1">
            <h3>
              {{ $t('detail.appeal.modal.evidenceInvalid.title') }}
              <Icon type="ios-alert-outline" color="red"/>
            </h3>
            <ol>
              <li>{{ $t('detail.appeal.modal.evidenceInvalid.1') }}</li>
              <li>{{ $t('detail.appeal.modal.evidenceInvalid.2') }}</li>
              <li>{{ $t('detail.appeal.modal.evidenceInvalid.3') }}</li>
              <li>{{ $t('detail.appeal.modal.evidenceInvalid.4') }}</li>
            </ol>
          </Col>
        </Row>
      </Card>
      <br>

      <Card dis-hover>
        <!-- 基本信息 -->
        <Form label-position="top">
          <Row :gutter="30">
            <Col flex="1">
              <FormItem :label="$t('detail.appeal.info.player')" prop="id">
                <Input type="text"
                       readonly
                       size="large"
                       :value="cheater.id"
                       :placeholder="$t('detail.placeholder.player')"/>
              </FormItem>
            </Col>
            <Col flex="1">
              <FormItem :label="$t('detail.appeal.info.originName')" prop="originName">
                <Input type="text"
                       :value="cheater.originName"
                       readonly
                       size="large"/>
              </FormItem>
            </Col>
          </Row>

          <!-- 下拉框，选择对申诉的操作 -->
          <FormItem :label="$t('detail.appeal.deal.type')">
            <Select v-model="appeal.type">
              <Option :value="i.value" :label="$t(i.name)" v-for="(i, index) in appealType" :key="index">
                <Row>
                  <Col flex="1">{{ $t(i.name) }}</Col>
                  <Col>{{ i.value }}</Col>
                </Row>
              </Option>
            </Select>
          </FormItem>
        </Form>

        <Form label-position="top" rel="moss_appeal" v-if="appeal.type === 'moss'"
              ref="detailAppealForm_moss"
              :model="appeal.fromData">
          <Row :gutter="30">
            <Col :xs="{span: 24}" :lg="{span: 12}">
              <!-- 第一人称录屏 -->
              <FormItem :label="$t('detail.appeal.deal.firstPersonRecording')"
                        :rules="{required: true, trigger: 'blur'}"
                        :prop="'extendedLinks.videoLink'">
                <EditLinks v-model="appeal.fromData.extendedLinks.videoLink"
                           :links="appeal.fromData.extendedLinks.btrLink"
                           :max="10"
                           :placeholder="$t('detail.appeal.placeholder.videolink')"></EditLinks>
              </FormItem>

              <!-- BTR链接 -->
              <FormItem :label="$t('detail.appeal.deal.btrLink')"
                        :rules="{required: true, trigger: 'blur'}"
                        :prop="'extendedLinks.btrLink'">
                <EditLinks v-model="appeal.fromData.extendedLinks.btrLink"
                           :links="appeal.fromData.extendedLinks.btrLink"
                           :max="100"
                           :placeholder="$t('detail.appeal.placeholder.btrlink')"></EditLinks>
              </FormItem>
            </Col>
            <Col :xs="{span: 24}" :lg="{span: 12}">
              <!-- MOSS上传按钮 -->
              <FormItem :label="$t('detail.appeal.deal.userGeneratedMossFile')"
                        :rules="{required: true, validator: checkAppealAttachmentsFile, trigger: 'change'}"
                        :prop="'extendedLinks.appendix'">
                <Upload multiple
                        type="drag"
                        action=""
                        accept="application/*"
                        :before-upload="handleFileMossUpload">
                  <div style="padding: 20px 0" :class="appeal.appendixStateStyle">
                    <Icon type="md-document" size="52"></Icon>
                  </div>
                </Upload>
              </FormItem>
            </Col>
          </Row>

          <!-- 玩家的申诉内容 -->
          <FormItem :label="$t('detail.appeal.info.content')"
                    :rules="{required: true, min: 10, trigger: 'change'}"
                    :prop="'content'">
            <Card dis-hover :padding="0">
                <Textarea v-model="appeal.fromData.content"
                          ref="moss_textareaAppealContent"
                          :toolbar="['bold', 'link']"
                          :height="'420px'"
                          :maxlength="40000"
                          :show-maxlength-label="true"
                          :placeholder="$t('detail.appeal.placeholder.content')"></Textarea>
            </Card>
          </FormItem>
        </Form>
        <Form label-position="top" rel="farm_appeal" v-if="appeal.type === 'farm'">
          <Row :gutter="30">
            <Col :xs="{span: 24}" :lg="{span: 12}">
              <!-- BTR链接 -->
              <FormItem :label="$t('detail.appeal.deal.btrLink')"
                        :rules="{required: true, trigger: 'blur'}"
                        :prop="'extendedLinks.btrLink'">
                <EditLinks v-model="appeal.fromData.extendedLinks.btrLink"
                           :links="appeal.fromData.extendedLinks.btrLink"
                           :max="100"
                           :placeholder="$t('detail.appeal.placeholder.btrlink')"></EditLinks>
              </FormItem>
            </Col>
          </Row>

          <!-- 玩家的申诉内容 -->
          <FormItem :label="$t('detail.appeal.info.content')"
                    :rules="{required: true, min: 10, trigger: 'change'}"
                    :prop="'content'">
            <Card dis-hover :padding="0">
                <Textarea v-model="appeal.fromData.content"
                          ref="farm_textareaAppealContent"
                          :toolbar="['bold', 'link']"
                          :height="'420px'"
                          :maxlength="40000"
                          :show-maxlength-label="true"
                          :placeholder="$t('detail.appeal.placeholder.content')"></Textarea>
            </Card>
          </FormItem>
        </Form>
        <Form label-position="top" rel="none_appeal" v-if="appeal.type === 'none'">
          <Alert show-icon type="error">
            此分类属于非常规申诉，在上方没有你所选对应类型时使用,在下方文本提交特殊材料，如果你属于'刷枪'或'自证'请重新更正申诉类型；使用不合适的申诉类型将导致案件被锁定，遭到管理永久无视此案件申诉
          </Alert>

          <!-- 玩家的申诉内容 -->
          <FormItem :label="$t('detail.appeal.info.content')"
                    :rules="{required: true, min: 10, trigger: 'change'}"
                    :prop="'content'">
            <Card dis-hover :padding="0">
                <Textarea v-model="appeal.fromData.content"
                          ref="none_textareaAppealContent"
                          :toolbar=" [[{'list': 'ordered'}, {'list': 'bullet'}], ['bold', 'hr'], ['link', 'image']]"
                          :height="'520px'"
                          :maxlength="60000"
                          :show-maxlength-label="true"
                          :placeholder="$t('detail.appeal.placeholder.content')"></Textarea>
            </Card>
          </FormItem>
        </Form>

        <Spin size="large" fix v-show="!isLogin">
          <div>
            <Icon type="md-lock" size="100"/>
          </div>
          <br>
          <Button :to="{name: 'signin'}">{{ $t("header.signin") }}</Button>
        </Spin>

        <Spin size="large" fix v-show="isLogin && cheater.status != 1 || cheater == null">
          <div>
            <Icon type="md-lock" size="100"/>
          </div>
          <br>
        </Spin>
      </Card>
      <br>

      <Row>
        <Col flex="1"></Col>
        <Col>
          <ButtonGroup>
            <Button size="large"
                    :to="{name: 'player',params: {ouid: $route.params.ouid} , query: {byPath: $route.name}}">{{
                $t('basic.button.cancel')
              }}
            </Button>
            <Button size="large" @click="resetAppealForm()">{{ $t('basic.button.reset') }}</Button>
          </ButtonGroup>
          <Divider type="vertical"/>
          <Button size="large" type="primary" :loading="appeal.load"
                  :disabled="!isLogin || cheater.status != 1 || cheater == null"
                  @click="handleAppeal">
            {{ $t('basic.button.commit') }}
          </Button>
        </Col>
      </Row>
    </div>
  </div>
</template>

<script>
import {api, application, http_token, message, moss, upload} from "@/assets/js";

import Empty from "@/components/Empty.vue";
import Textarea from "@/components/textarea/index.vue";
import Captcha from "@/components/captcha/index";
import Html from "@/components/Html.vue";
import EditLinks from "@/components/EditLinks.vue"

export default new application({
  name: "detilaPlayerAppeal",
  data() {
    return {
      appealType: [
        {
          'name': 'detail.appeal.deal.moss',
          'value': 'moss'
        },
        {
          'name': 'detail.appeal.deal.farm',
          'value': 'farm',
        },
        {
          'name': 'detail.appeal.deal.other',
          'value': 'none',
        }
      ],
      appeal: {
        load: false,
        fromData: {
          extendedLinks: {
            videoLink: '',
            btrLink: '',
            mossDownloadUrl: '',
          },
          content: '',
        },
        disable: this.$store.state.configuration.detailLeftAppealPanel ?? false,
        appendix: null,
        appendixStateStyle: '',
        toPlayerId: 0,
        type: '',
      },
      cheater: {},
    }
  },
  components: {
    Empty,
    Textarea,
    Captcha,
    Html,
    EditLinks
  },
  watch: {
    '$route': 'loadData',
  },
  created() {
    this.http = http_token.call(this);

    this.appeal.type = this.appealType[0].value;

    this.loadData();
  },
  methods: {
    async loadData() {
      // set Token Http mode
      this.http = http_token.call(this);

      await this.getPlayerInfo();
    },
    /**
     * 获取举报玩家档案
     */
    async getPlayerInfo() {
      return new Promise(resolve => {
        let params = {
          history: true,
          personaId: this.$route.params.ouid
        };

        // 旧网站URL, 兼容
        if (this.$route.query.oldUrl && this.$route.params.ouid) {
          params = {
            history: true,
            userId: this.$route.params.ouid
          };
          delete params.personaId;
        }

        this.cheater = {};

        this.http.get(api["player"], {params}).then(res => {
          const d = res.data;

          if (d.success === 1) {
            // 历史名称排序
            d.data.history = d.data.history.sort(function (a, b) {
              let aTime = new Date(a.fromTime).getTime();
              let bTime = new Date(b.fromTime).getTime();
              return aTime > bTime ? 1 : -1;
            })

            this.cheater = d.data;
            return;
          }

          switch (d.code) {
            case "player.bad":
            case "player.notFound":
              this.$router.push({name: 'player_list'})
              break;
          }

          this.$Message.info(this.$t('basic.tip.notFound'));
        }).finally(() => {
          resolve()
        });
      })
    },
    /**
     * 校验附件(MOSS)
     */
    checkAppealAttachmentsFile(rule, value, callback) {
      const val = this.appeal.appendix;
      const isMossMode = this.appeal.type == 'moss';

      if (isMossMode && !val) return callback('not file');
      if (isMossMode && this.appeal.appendixStateStyle == 'ivu-alert-warning') return callback('Non compliant attachments');

      callback()
    },
    /**
     * 选择Moss文件
     * @param event
     */
    handleFileMossUpload(file) {
      const that = this;
      that.clearFileMoss();

      moss.verifyFileIsMoss(file).then(res => {
        if (res.code == 0) {
          that.appeal.appendixStateStyle = 'ivu-alert-success';
          that.appeal.appendix = file;
          that.$Message.success(res.message);
        } else {
          that.appeal.appendixStateStyle = 'ivu-alert-warning';
          if (res.message) that.$Message.error(res.message);
        }
      })
    },
    /**
     * 清除Moss File选择的数据
     */
    clearFileMoss() {
      this.appeal.appendix = null;
      this.appeal.appendixStateStyle = '';
    },
    /**
     * 提交申诉
     * @returns {boolean}
     */
    handleAppeal() {
      try {
        const type = this.appeal.type;
        const content = this.appeal.content || this.$refs[`${type}_textareaAppealContent`].editorContent;

        this.appeal.load = true;

        // 验证表单
        this.$refs[`detailAppealForm_${type}`].validate(async (valid) => {
          if (!valid) return;

          let appealDataFormData = {
            data: {
              data: {
                toPlayerId: this.cheater.id,
                appealType: this.appeal.type,
                content: content
              }
            }
          };

          switch (type) {
            case 'moss':
              appealDataFormData.data.data = Object.assign(appealDataFormData.data.data, {
                extendedLinks: {
                  videoLink: this.appeal.fromData.extendedLinks.videoLink,
                  btrLink: this.appeal.fromData.extendedLinks.btrLink
                }
              });

              await upload.on(this.appeal.appendix)
                  .then(res => {
                    if (res && res.code >= 1) {
                      appealDataFormData.data.data = Object.assign(appealDataFormData.data.data, {
                        extendedLinks: {
                          mossDownloadUrl: res.filename
                        }
                      });
                      return;
                    }

                    this.$Message.error(res.message || res.code);
                  })
                  .catch(err => {
                    this.$Message.error(err.message || err.code);
                  })
                  .finally(() => {
                    this.appeal.load = false;
                    message.playSendVoice();
                  });
              break;
            case 'farm':
              appealDataFormData.data.data = Object.assign(appealDataFormData.data.data, {
                extendedLinks: {
                  btrLink: this.appeal.fromData.extendedLinks.btrLink
                }
              });
              break;
            case 'none':
            default:
              // No additional data for 'none' type
          }

          // Then, submit the appeal
          const res = await this.http.post(api["player_banAppeal"], appealDataFormData);
          const d = res.data;

          if (d.success === 1) {
            this.$Message.success(this.$t(`basic.tip['${d.code}']`));
            this.appeal.show = false;
          } else {
            this.$Message.error(this.$t(`basic.tip['${d.code}']`, {
              message: d.message || ""
            }));
          }
        })

      } catch (error) {
        this.$Message.error(error.message || error.code);
        this.$emit('success');
      } finally {
        this.appeal.load = false;
        message.playSendVoice();
      }

      return false;
    },
    /**
     * 重置表单
     */
    resetAppealForm() {
      const that = this;
      this.clearFileMoss();
      this.appealType.forEach(i => {
        const appeal = `${i["value"]}_appeal`;
        const textarea = `${i["value"]}_textareaAppealContent`;
        if (that.$refs[appeal])
          that.$refs[appeal].resetFields();
        if (that.$refs[textarea])
          that.$refs[textarea].updateContent("");
      })
    }
  },
})
</script>

<style scoped lang="less">

</style>
