<template>
  <div class="container">
    <div class="content">
      <br>
      <Row>
        <Col :xs="{push: 1}" :lg="{push: 0}">
          <Breadcrumb>
            <BreadcrumbItem :to="{name: 'home'}">{{ $t("header.index") }}</BreadcrumbItem>
            <BreadcrumbItem>{{ $t("forgetPassword.title") }}</BreadcrumbItem>
          </Breadcrumb>
        </Col>
      </Row>
      <br>

      <Row>
        <Col span="24">
          <Card :padding="0" dis-hover>
            <Row>
              <Col :mg="{span: 0}" :lg="{span: 8}" v-show="!isMobile">
                <Banner :style="`min-height:600px; height: 100%`" :show-align="'upDown'" :height="600">
                  <Row style="padding: 20px" type="flex" align="middle">
                    <Col>
                      <Alert show-icon><span v-html="$t('forgetPassword.forgetUsername')"></span></Alert>
                    </Col>
                  </Row>
                </Banner>
              </Col>
              <Col flex="1">
                <Card :bordered="false" dis-hover>
                  <Steps :current="stepsIndex" slot="title">
                    <Step :title="$t('signup.steps[0].title')" :content="$t('signup.steps[0].title')"></Step>
                    <Step :title="$t('signup.steps[2].title')" :content="$t('signup.steps[2].title')"></Step>
                    <Step :title="$t('signup.steps[3].title')" :content="$t('signup.steps[3].title')"></Step>
                    <Step :title="$t('signup.steps[4].title')" :content="$t('signup.steps[4].title')"></Step>
                  </Steps>

                  <div>
                    <Form ref="formValidate" label-position="top" :rules="ruleValidate" style="position: relative;">
                      <div v-if="stepsIndex == 0">
                        <FormItem :label="$t('forgetPassword.form.username')" prop="username">
                          <Input v-model="forgetPassword.username" size="large"
                                 :placeholder="$t('forgetPassword.placeholder.username')"/>
                        </FormItem>
                        <FormItem :label="$t('forgetPassword.form.originEmail')" prop="originEmail">
                          <Input v-model="forgetPassword.originEmail" size="large"
                                 :placeholder="$t('forgetPassword.placeholder.originEmail')"/>
                        </FormItem>
                      </div>

                      <FormItem v-if="stepsIndex == 1" :label="$t('captcha.title')">
                        <Captcha ref="captcha" @getCaptchaData="getCaptchaData"></Captcha>
                      </FormItem>

                      <div v-if="stepsIndex == 2">
                        <EmailTip :email="forgetPassword.originEmail"></EmailTip>
                        <Card dis-hover>
                          <Row :gutter="16" type="flex" justify="center" align="middle">
                            <Col>
                              <Icon type="md-cloud" color="#535353" size="80"/>
                            </Col>
                            <Col>
                              <Icon type="md-return-right" color="#aaa" size="30"/>
                            </Col>
                            <Col>
                              <Icon type="md-mail" color="#535353" size="80"/>
                            </Col>
                          </Row>
                        </Card>
                        <br>
                        <Alert type="success" show-icon>{{ $t('forgetPassword.checkEmail') }}</Alert>
                      </div>

                      <div v-if="stepsIndex == 3">
                        <FormItem :label="$t('reset.form.newPassword')" prop="password">
                          <Input type="password" password minlength="6" v-model="forgetPassword.password" size="large"
                                 :placeholder="$t('signup.placeholder.password')"/>
                        </FormItem>
                      </div>

                      <!-- 重置成功 S -->
                      <div v-if="stepsIndex == 4" align="center">
                        <h1>{{ $t('forgetPassword.resetSuccess') }}</h1>
                      </div>
                      <!-- 重置成功 E -->

                    </Form>
                  </div>
                </Card>
              </Col>
            </Row>

            <template>
              <Affix :offset-bottom="0" :useCapture="true" style="margin: 0px">
                <Divider size="small" plain style="margin: 0px"></Divider>

                <Card dis-hover :padding="8" :bordered="false">
                  <Row>
                    <Col flex="1"></Col>
                    <Col span="12" align="right" type="flex">
                      <Button v-if="button.prev"
                              :disabled="button.prevShow"
                              @click.prevent.stop="stepsIndex--; onStepsIndex();" size="large">{{
                          $t('basic.button.prev')
                        }}
                      </Button>
                      <Divider type="vertical"/>
                      <Button v-if="button.next"
                              :disabled="button.nextShow"
                              @click.prevent.stop="stepsIndex++; onStepsIndex();" size="large"
                              type="primary">{{ $t('basic.button.next') }}
                      </Button>
                      <Button v-if="button.submit"
                              @click.prevent.stop="onSubmit"
                              :loading="spinShow"
                              size="large" type="primary">
                        {{ $t('basic.button.submit') }}
                      </Button>
                    </Col>
                  </Row>
                </Card>
              </Affix>
            </template>
          </Card>
        </Col>
      </Row>
    </div>
  </div>
</template>

<script>
import {api, application, http, mail} from "@/assets/js";

import EmailTip from "@/components/EmailTip";
import Captcha from "@/components/captcha/index";
import Banner from "@/components/Banner"

export default new application({
  name: 'userForgetPassword',
  data() {
    return {
      button: {
        next: true,
        nextShow: false,
        prev: true,
        prevShow: true,
        submit: false
      },
      stepsIndex: 0,
      ruleValidate: {},
      forgetPassword: {
        username: '',
        originEmail: '',
        captcha: {}
      },
      spinShow: false,

      verify: {
        load: 0,
      },
    }
  },
  components: {EmailTip, Captcha, Banner},
  created() {
    const {code} = this.$route.query;
    if (code) {
      this.stepsIndex = 3;
      this.onStepsIndex();
    }
  },
  methods: {
    onStepsIndex() {
      let stepsIndex = this.stepsIndex;
      this.button.submit = false;

      switch (stepsIndex) {
        case 0:
          this.button.prev = true;
          this.button.prevShow = true;
          this.button.next = true;
          break;
        case 1:
          this.button.prev = true;
          this.button.prevShow = false;
          this.button.next = false;
          this.button.submit = true;
          break;
        case 2:
          this.button.prev = false;
          this.button.next = false;
          break;
        case 3:
          this.button.prev = false;
          this.button.next = false;
          this.button.submit = true;
          break;
        case 4:
          this.button = {};
          break;
      }

      return this.stepsIndex;
    },

    getCaptchaData(value) {
      this.forgetPassword.captcha = value;
    },

    /**
     * 提交
     */
    onSubmit() {
      switch (this.stepsIndex) {
        case 1:
          this.onForgetPassword();
          break;
        case 3:
          this.forgetPasswordVerify();
          break;
      }
    },

    /**
     * 重置密码 key验证
     * @param code
     */
    forgetPasswordVerify() {
      const {code} = this.$route.query;
      const newpassword = this.forgetPassword.password;
      this.spinShow = true;

      if (code == '' || code == undefined || code == null) {
        this.verify.iscode = false;
        return;
      }
      this.verify.iscode = true;
      this.verify.load = 0;

      http.post(api["user_forgetPasswordVerify"], {
        data: {
          data: {
            code,
            newpassword
          }
        }
      }).then(res => {
        const d = res.data;

        if (d.success === 1) {
          this.verify.load = 3;
          this.stepsIndex = 4;
          this.$Message.success(this.$t(`basic.tip['${d.code}']`, {
            message: d.message || ""
          }));
          return;
        }

        this.verify.load = -1;
        this.$Message.error(this.$t(`basic.tip['${d.code}']`, {
          message: d.message || ""
        }));
      }).catch((e) => {
        this.verify = {
          load: -1,
          msg: e.toString(),
        };
      }).finally(() => {
        this.spinShow = false;
        this.onStepsIndex();
      });
    },

    /**
     * 提交 重置密码
     */
    onForgetPassword() {
      this.spinShow = true;

      const {username, originEmail, captcha} = this.forgetPassword;

      http.post(api["user_forgetPassword"], {
        data: {
          data: Object.assign({username, originEmail}, {language: mail.exchangeLangField(this.$root.$i18n.locale)}),
          captcha,
        },
      }).then(res => {
        const d = res.data;

        if (d.success === 1) {
          this.stepsIndex++;
          this.$Message.success(this.$t(`basic.tip['${d.code}']`, {
            message: d.message || ""
          }));
          return;
        }

        this.$Message.error(this.$t(`basic.tip['${d.code}']`, {
          message: d.message || ""
        }));
      }).catch(err => {
        this.$Message.error(err.toString());
      }).finally(() => {
        this.onStepsIndex();
        this.spinShow = false;
      });
    },
  },
});
</script>

<style lang="less" scoped>

</style>
