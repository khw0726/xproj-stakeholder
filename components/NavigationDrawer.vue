<template>
  <v-navigation-drawer
    fixed
    clipped
    class="grey lighten-4"
    app
    v-model="drawerFlag"
    temporary
    >
    <v-list class="pa-1">
      <v-list-tile avatar>
        <v-list-tile-avatar>
          <v-icon>person</v-icon>
        </v-list-tile-avatar>

        <v-list-tile-content>
          <v-list-tile-title>{{userEmail}}</v-list-tile-title>
        </v-list-tile-content>
      </v-list-tile>
    </v-list>

    <v-divider/>

    <template v-if="!experimentDone">
      <v-list class="pt-0" dense>

        <v-list-group
          no-action
          sub-group
          value="true"
          >
          <v-list-tile slot="activator">
            <v-list-tile-action>
              <v-icon>query_builder</v-icon>
            </v-list-tile-action>

            <v-list-tile-content>
              <v-list-tile-title>실험 진행상황</v-list-tile-title>
            </v-list-tile-content>
          </v-list-tile>

          <v-list-tile>
            <v-list-tile-title>정책</v-list-tile-title>
            <v-list-tile-action>
              <span v-if="step>=2" style="color:blue;">{{step}} / 2</span>
              <span v-else style="color:red;">{{step}} / 2</span>
            </v-list-tile-action>
          </v-list-tile>

          <v-list-tile>
            <v-list-tile-title>영향 입력</v-list-tile-title>
            <v-list-tile-action>
              <span v-if="stakeholders_answered>=stakeholders_answered_min" style="color:blue;">{{stakeholders_answered}} / {{stakeholders_answered_min}}</span>
              <span v-else style="color:red;">{{stakeholders_answered}} / {{stakeholders_answered_min}}</span>
            </v-list-tile-action>
          </v-list-tile>

          <v-list-tile>
            <v-list-tile-title>(영향 읽기) 태그 수</v-list-tile-title>
            <v-list-tile-action>
              <span v-if="effects_seen>=effects_seen_min" style="color:blue;">{{effects_seen}} / {{effects_seen_min}}</span>
              <span v-else style="color:red;">{{effects_seen}} / {{effects_seen_min}}</span>
            </v-list-tile-action>
          </v-list-tile>

        </v-list-group>

        <v-list-group
          no-action
          sub-group
          value="true"
          >
          <v-list-tile slot="activator">
            <v-list-tile-action>
              <v-icon>query_builder</v-icon>
            </v-list-tile-action>

            <v-list-tile-content>
              <v-list-tile-title>현재 단계</v-list-tile-title>
            </v-list-tile-content>
          </v-list-tile>
        <v-container>
          <v-layout row wrap>
            <v-flex xs4 v-for="n in 3" :key="n">
              <v-card v-if="n<currentStep" dark color="blue" v-html="instr_step[n-1]"></v-card>
              <v-card v-else-if="n===currentStep" id="currentStep_outline"> 
                <strong style="color:green;" v-html="instr_step[n-1]"></strong> 
              </v-card>
              <v-card v-else v-html="instr_step[n-1]"></v-card>
            </v-flex>
          </v-layout>
        </v-container>
        </v-list-group>
      </v-list>

      <v-divider/>
      <v-list class="pt-0" dense>

        <v-list-tile @click="onShowDropDialog">
          <v-list-tile-action>
            <v-icon>pan_tool</v-icon>
          </v-list-tile-action>

          <v-list-tile-content>
            <v-dialog
              v-model="dropDialog"
              width="500"
              full-width
            >
              <v-list-tile-title slot="activator">포기하기</v-list-tile-title>

              <v-card>
                <v-card-title
                  class="headline grey lighten-2 warning"
                  color="red"
                  primary-title
                > 주의
                </v-card-title>

                <v-card-text>
                  <h2 strong style="color:red;"> 정말 포기하시겠습니까? </h2>
                  실험 도중 포기는 실험 참가자의 고유한 권리입니다.<br>
                  지금 포기하시면 여태까지 시스템에 입력하셨던 사용자의 정보가 안전히 폐기되며, 처음으로 돌아가게 됩니다.

                </v-card-text>

                <v-divider></v-divider>

                <v-card-actions>
                  <v-btn
                    color="primary"
                    flat outline ripple
                    @click="onCloseDropDialog"
                  > 돌아가기 </v-btn>
                  <v-spacer></v-spacer>
                  <v-btn
                    color="red"
                    flat outline ripple
                    @click="giveUp"
                  >
                    포기하기
                  </v-btn>
                </v-card-actions>

              </v-card>

            </v-dialog>
          </v-list-tile-content>
        </v-list-tile>  

        <v-list-tile @click="tutorial">
          <v-list-tile-action>
            <v-icon>directions</v-icon>
          </v-list-tile-action>

          <v-list-tile-content>
            <v-list-tile-title>튜토리얼</v-list-tile-title>
          </v-list-tile-content>
        </v-list-tile>  
      </v-list>
    </template>

    <template v-if="isNotLoggedIn">
      <v-list class="pt-0" dense>
        <v-list-tile @click="toLogIn">
          <v-list-tile-action>
            <v-icon>input</v-icon>
          </v-list-tile-action>

          <v-list-tile-content>
            <v-list-tile-title>로그인 하러 가기</v-list-tile-title>
          </v-list-tile-content>
        </v-list-tile>
      </v-list>
    </template>

    <template v-if="experimentDone"> <!-- guest -->
      <v-list class="pt-0" dense>
        <v-list-tile @click="tutorial">
          <v-list-tile-action>
            <v-icon>directions</v-icon>
          </v-list-tile-action>

          <v-list-tile-content>
            <v-list-tile-title>튜토리얼</v-list-tile-title>
          </v-list-tile-content>
        </v-list-tile>
        <v-list-tile @click="showPolicies">
          <v-list-tile-action>
            <v-icon>list</v-icon>
          </v-list-tile-action>

          <v-list-tile-content>
            <v-list-tile-title>정책 목록 보기</v-list-tile-title>
          </v-list-tile-content>
        </v-list-tile>  
      </v-list>
    </template>


    <v-list class="pt-0" dense>
      <v-list-tile @click="onShowAboutDialog">
        <v-list-tile-action>
          <v-icon>info</v-icon>
        </v-list-tile-action>
        
        <v-list-tile-content>
          <v-dialog
            v-model="aboutDialog"
            width="500"
            full-width
            >
            <v-list-tile-title slot="activator" @click="onShowAboutDialog">연락처</v-list-tile-title> <!-- TODO: GA-->
              <v-card>
                <v-card-title class="headline grey lighten-2" primary-title>
                  About
                </v-card-title>
                <v-card-text>
                  사용하시면서 불편하신 점이나 궁금하신 점이 있다면, <a href="mailto:xproject@kixlab.org">xproject@kixlab.org</a>로 연락주세요.
                  <br>
                  본 연구는 <a @click="linkToKAISTSoC"> KAIST 전산학부 </a> <a @click="linkToKIXLAB">인터랙션 연구실</a>에서 수행 중인 한국연구재단의 “소셜컴퓨팅 기술을 활용한 국가청렴도 개선” 과제의 일환입니다.<br><br>

                </v-card-text>
                <v-divider></v-divider>
                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn color="primary" flat @click.native="onCloseAboutDialog">
                    닫기
                  </v-btn>
                </v-card-actions>
            </v-card>
          </v-dialog>
        </v-list-tile-content>
      </v-list-tile>  

      <v-flex>
        <br>
      </v-flex>

      <v-list-tile @click="drawerFlag = false">
        <v-list-tile-action>
          <v-icon>keyboard_arrow_left</v-icon>
        </v-list-tile-action>

        <v-list-tile-content>
          <v-list-tile-title><strong>닫기</strong></v-list-tile-title>
        </v-list-tile-content>
      </v-list-tile>

    </v-list>
  </v-navigation-drawer>
</template>

<style scoped>
.warning {
  background-color: pink !important;
}
#currentStep_outline {
  outline-width: 40px;
  outline: auto;
  outline-color: green;
}
</style>

<script>
export default {
  computed: {
    currentStep: function () {
      switch (this.$router.currentRoute.path) {
        case '/ReadNews': {
          return 1
        }
        case '/Identify':
        case '/StateAsStakeholder': {
          return 2
        }
        case '/GuessEffectRandom':
        case '/VerifyEffect': {
          return 3
        }
        case '/TagOverview':
        case '/ExploreOpinions': {
          return 4
        }
        default: {
          return 0
        }
      }
    },
    user: function () {
      return this.$store.state.user
    },
    isNotLoggedIn: function () {
      return !this.$store.state.userToken
    },
    userEmail: function () {
      if (this.user.email != null) {
        return this.user.email
      }
      return '둘러보는 중'
    },
    userGroup: function () {
      // if (!this.$store.state.user.is_participant) {
      //   return -1
      // } else {
      //   return this.$store.getters.experimentCondition
      // }
      return this.$store.getters.userGroup
    },
    step: function () {
      const userstep = this.$store.state.user.step
      if (userstep === 0) {
        return 0
      }
      return userstep - 1
    },
    stakeholders_answered: function () {
      return this.$store.state.userPolicy.stakeholders_answered
    },
    // stakeholders_seen: function () {
    //   return this.$store.state.userPolicy.stakeholders_seen
    // },
    effects_seen: function () {
      return this.$store.state.userPolicy.effects_seen
    },
    stakeholders_answered_min: function () {
      switch (this.userGroup) {
        case 1:
        case 2:
        case 3:
        case 4:
        case -1:
          return 0
        case 5:
        case 0:
        case 6:
        case 7:
          return 3
      }
    },
    // stakeholders_seen_min: function () {
    //   switch (this.userGroup) {
    //     case 1:
    //     case 2:
    //     case -1:
    //       return 0
    //     case 3:
    //     case 4:
    //     case 5:
    //     case 0:
    //       return 3
    //   }
    // },
    effects_seen_min: function () {
      switch (this.userGroup) {
        case 1:
        case 2:
        case -1:
        case 6:
        case 7:
          return 0
        case 3:
        case 4:
        case 5:
        case 0:
          return 9
      }
    },
    experimentDone: function () {
      return !this.$store.state.userToken || !this.user.is_participant || this.user.step >= 3
    },
    drawerFlag: {
      get: function () {
        return this.drawer
      },
      set: function (val) {
        this.$emit('emitterdrawer', val)
      }
    },
    instr_step: function () {
      switch (this.userGroup) {
        case 1:
        case 2:
          return ['뉴스<br>읽기', ' ', ' ']
        case 3:
        case 4:
          return ['뉴스<br>읽기', ' ', '영향<br>읽기']
        case 6:
        case 7:
          return ['뉴스<br>읽기', '영향<br>말하기', ' ']
        case 5:
        case 0:
          return ['뉴스<br>읽기', '영향<br>말하기', '영향<br>읽기']
        case -1:
          return [' ', ' ', ' ', ' ']
      }
    }
  },
  props: {
    drawer: Boolean,
    source: String
  },
  methods: {
    goHome: function () {
      this.$ga.event({
        eventCategory: this.$router.currentRoute.path,
        eventAction: 'GoHome',
        eventLabel: (this.$store.state.policy && this.$store.state.policy.title) ? this.$store.state.policy.title : '',
        eventValue: 0
      })
      this.$router.push('/')
    },
    showPolicies: function () {
      this.$ga.event({
        eventCategory: this.$router.currentRoute.path,
        eventAction: 'ToShowPolicies',
        eventLabel: (this.$store.state.policy && this.$store.state.policy.title) ? this.$store.state.policy.title : '',
        eventValue: 0
      })
      this.$router.push('/ShowPolicies')
    },
    toLogIn: function () {
      this.$ga.event({
        eventCategory: this.$router.currentRoute.path,
        eventAction: 'ToLogin',
        eventLabel: (this.$store.state.policy && this.$store.state.policy.title) ? this.$store.state.policy.title : '',
        eventValue: 0
      })
      this.$router.push('/SignIn')
    },
    giveUp: function () {
      this.$ga.event({
        eventCategory: this.$router.currentRoute.path,
        eventAction: 'GiveUp',
        eventLabel: (this.$store.state.policy && this.$store.state.policy.title) ? this.$store.state.policy.title : '',
        eventValue: 0
      })
      this.$store.commit('logout')
      this.$router.push('/')
    },
    tutorial: function () {
      this.$ga.event({
        eventCategory: this.$router.currentRoute.path,
        eventAction: 'SeeTutorial',
        eventLabel: (this.$store.state.policy && this.$store.state.policy.title) ? this.$store.state.policy.title : '',
        eventValue: 0
      })
      this.$router.push('/Tutorial')
    },
    onShowDropDialog: function () {
      this.$ga.event({
        eventCategory: this.$router.currentRoute.path,
        eventAction: 'ShowDropDialog',
        eventLabel: (this.$store.state.policy && this.$store.state.policy.title) ? this.$store.state.policy.title : '',
        eventValue: 0
      })
      this.dropDialog = true
    },
    onCloseDropDialog: function () {
      this.$ga.event({
        eventCategory: this.$router.currentRoute.path,
        eventAction: 'SeeTutorial',
        eventLabel: (this.$store.state.policy && this.$store.state.policy.title) ? this.$store.state.policy.title : '',
        eventValue: 0
      })
      this.dropDialog = false
    },
    onShowAboutDialog: function () {
      this.$ga.event({
        eventCategory: this.$router.currentRoute.path,
        eventAction: 'ShowAboutDialog',
        eventLabel: (this.$store.state.policy && this.$store.state.policy.title) ? this.$store.state.policy.title : '',
        eventValue: 0
      })
      this.aboutDialog = true
    },
    onCloseAboutDialog: function () {
      this.$ga.event({
        eventCategory: this.$router.currentRoute.path,
        eventAction: 'CloseAboutDialog',
        eventLabel: (this.$store.state.policy && this.$store.state.policy.title) ? this.$store.state.policy.title : '',
        eventValue: 0
      })
      this.aboutDialog = false
    },
    linkToKAISTSoC: function () {
      this.$ga.event({
        eventCategory: this.$router.currentRoute.path,
        eventAction: 'OpenKAISTSoC',
        eventLabel: (this.$store.state.policy && this.$store.state.policy.title) ? this.$store.state.policy.title : '',
        eventValue: 0
      })
      window.open('http://cs.kaist.ac.kr', '_blank')
    },
    linkToKIXLAB: function () {
      this.$ga.event({
        eventCategory: this.$router.currentRoute.path,
        eventAction: 'OpenKIXLAB',
        eventLabel: (this.$store.state.policy && this.$store.state.policy.title) ? this.$store.state.policy.title : '',
        eventValue: 0
      })
      window.open('https://kixlab.org', '_blank')
    }
  },
  data: function () {
    return {
      dropDialog: false,
      aboutDialog: false
    }
  }
}
</script>


