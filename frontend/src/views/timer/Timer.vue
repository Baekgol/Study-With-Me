<template>
  <div>
    <v-dialog v-model="dialog" width="700">
      <v-card>
        <div style="float:right;padding:10px;">
          <v-icon @click="dialog = !dialog">mdi-close</v-icon>
        </div>
        <v-card-title style="text-align:center;">
          <v-img src="@/assets/img/logo_bl.png" max-width="200"></v-img>
          <span id="tutorialTitle">어떻게 사용해요?</span>
        </v-card-title>
        <v-carousel v-model="step" height="400">
          <v-carousel-item
            v-for="tutorial in tutorials"
            :key="tutorial.idx"
            :src="tutorial.src"
          >
          </v-carousel-item>
        </v-carousel>
        <v-card-text
          ><p id="tutorialContent">
            {{ tutorials[step].content }}
          </p></v-card-text
        >
      </v-card>
    </v-dialog>

    <div style="width:100%;">
      <v-container class="outbox">
        <v-row no-gutters>
          <!-- 왼쪽 스트리밍 화면 -->
          <v-col cols="9">
            <v-progress-circular
              v-if="!$store.state.isStartCam"
              class="loadingCircular"
              indeterminate
              color="#673fb4"
            ></v-progress-circular>
            <ImageStream
              v-show="$store.state.isStartCam"
              @startCam="startTimer"
              @runningTimer="runningTimer"
              @pauseTimer="pauseTimer"
              @resumeTimer="resumeTimer"
              @awayState="away = true"
              @phoneState="phone = true"
              @sleepState="sleep = true"
              @talkState="talk = true"
              @awayEnd="away = false"
              @phoneEnd="phone = false"
              @sleepEnd="sleep = false"
              @talkEnd="talk = false"
            />
            <v-img class="timerLogo" src="@/assets/img/logo_bl.png"></v-img>
            <v-btn
              class="timerTutorial"
              @click="openTutorial"
              color="#673fb4"
              style="color:white;"
              >튜토리얼</v-btn
            >
            <div id="detectTable">
              <v-expansion-panels accordion>
                <v-expansion-panel
                  style="background-color: rgba(255,255,255,0.5); border-radius: 20px; min-width: 230px;"
                >
                  <v-expansion-panel-header
                    style="font-size: 1.2rem; letter-spacing: -2px; "
                  >
                    방해요소 보기
                  </v-expansion-panel-header>
                  <v-expansion-panel-content style="letter-spacing: -1px;">
                    <v-row
                      no-gutters
                      style="letter-spacing: -1px; font-size: 1.1rem; font-weight: bolder;"
                    >
                      나는 지금까지..
                    </v-row>
                    <v-row
                      no-gutters
                      style="letter-spacing: -1px; font-size: 0.8rem; font-weight: ligter;"
                    >
                      <v-col>
                        <v-row no-gutters>
                          <v-col
                            style="margin-right: 5px; font-weight: bold;"
                            align="end"
                            >자리비움 {{ $store.getters.getAwayTime }}회</v-col
                          >
                          <v-col
                            style="margin-right: 5px; font-weight: bold;"
                            align="end"
                            >핸드폰 {{ $store.getters.getPhoneTime }}회</v-col
                          >
                        </v-row>
                        <v-row no-gutters>
                          <v-col
                            style="margin-right: 5px; font-weight: bold;"
                            align="end"
                            >졸음 {{ $store.getters.getSleepTime }}회</v-col
                          >
                          <v-col
                            style="margin-right: 5px; font-weight: bold;"
                            align="end"
                            >잡담 {{ $store.getters.getTalkTime }}회</v-col
                          >
                        </v-row>
                      </v-col>
                    </v-row>
                    <hr
                      style="margin-top: 10px; margin-bottom: 10px; background-color: white;"
                    />
                    <div
                      style="letter-spacingL -1px; font-weight: bolder; font-size: 1.2rem; color: red;"
                    >
                      <v-row v-if="away" no-gutters justify="center">
                        자리비움
                      </v-row>
                      <v-row v-if="phone" no-gutters justify="center">
                        핸드폰
                      </v-row>
                      <v-row v-if="sleep" no-gutters justify="center">
                        졸음
                      </v-row>
                      <v-row v-if="talk" no-gutters justify="center">
                        잡담
                      </v-row>
                    </div>
                  </v-expansion-panel-content>
                </v-expansion-panel>
              </v-expansion-panels>
            </div>
            <v-row
              no-gutters
              class="time-bar timer"
              justify="center"
              align="center"
            >
              {{ time }}
            </v-row>
          </v-col>

          <!-- 오른쪽 투두리스트 화면 -->
          <v-col cols="3">
            <div style="background-color: #EEEEEE;">
              <!-- 회원정보 -->
              <div>
                <div class="user">
                  <ProfileNormal
                    class="profile"
                    :src="'data:image/png;base64,' + profileImg"
                  />
                </div>
                <div class="user">
                  <span
                    ><strong>{{ user.userNickname }}</strong></span
                  ><br />
                  <span>{{ user.userMessage }}</span>
                </div>
              </div>
              <!-- 투두리스트 -->
              <div>
                <p class="todolist">TO DO LIST</p>
              </div>
              <v-card style="border-radius:60%;">
                <p class="date">{{ curruentDate }}</p>
                <TodoList
                  @updateTodoList="updateTodoList"
                  :todoList="todoList"
                  :date="todayis"
                />
                <br />
                <div @click="stopStudy" style="text-align:center;">
                  <AppBtnLarge
                    :btnColor="`#424242`"
                    :btnName="'그만하기'"
                    :btnNameColor="'white'"
                  />
                </div>
                <br />
              </v-card>
            </div>
          </v-col>
        </v-row>
      </v-container>
    </div>
  </div>
</template>

<script>
import ImageStream from './ImageStream.vue';
import TodoList from '@/components/common/TodoList.vue';
import ProfileNormal from '@/components/common/ProfileNormal.vue';
import AppBtnLarge from '@/components/common/AppBtnLarge.vue';
import date from '@/date.js';

const storage = window.sessionStorage;

export default {
  components: {
    ImageStream,
    TodoList,
    ProfileNormal,
    AppBtnLarge,
  },
  data: function() {
    return {
      dialog: false,
      step: 0,
      isLoading: true,
      tutorials: [
        {
          idx: 1,
          src: 'https://ifh.cc/g/Z2hjFW.jpg',
          content:
            '1. 불필요한 것에 신경쓰지 않고 공부만 할 수 있습니다. SWITHME는 얼굴과 행동,사물을 분석하여 집중/비집중 여부를 파악해주니까요! 하단에는 집중한 시간이 표시됩니다.',
        },
        {
          idx: 2,
          src: 'https://ifh.cc/g/zmJZj8.jpg',
          content:
            "2. 필요하다면 TO DO LIST를 활용해보세요. 상단에서 TO DO LIST를 추가하고, 완료하거나 삭제할 수 있습니다. 지난 투두리스트는 '나의학습'에서 확인할 수 있습니다.",
        },
        {
          idx: 3,
          src: 'https://ifh.cc/g/A5gphR.jpg',
          content:
            '3. 하단에는 투두리스트를 기준으로 현재 진행률이 표시됩니다. 학습을 종료하고 싶다면 그만하기 버튼을 눌러주세요. 그만하기 버튼을 누르면 메인 홈으로 돌아갑니다.',
        },
      ],
      todayis: '',
      isStudying: false,
      user: {},
      todoList: [],
      curruentDate: '',
      profileImg: '',
      tutorialFlag: false,
      time: '00:00:00',
      UTCminute: 0,
      UTCsecond: 0,
      accumulatedSec: 0,
      timeBegan: null,
      timeStopped: null,
      stoppedDuration: 0,
      // started: null,
      // running: false,
      hour: 0,
      min: 0,
      sec: 0,
      away: false,
      phone: false,
      sleep: false,
      talk: false,
    };
  },
  created() {
    this.$store.commit('setIsStartCam', false);
    this.getUserInfo();
    this.getUserTimer();
    this.getTodoList();
    let today = new Date();
    this.todayis = date.dateFunc(today);
  },
  methods: {
    getUserInfo() {
      this.$Axios
        .get('user', {
          headers: {
            'jwt-auth-token': storage.getItem('jwt-auth-token'),
          },
        })
        .then((res) => {
          if (res.data.data.userId) {
            console.log('로그인 정보 잘 받아와짐');
            this.user = res.data.data;
            this.profileImg = res.data.profileImg;
          } else {
            console.log('로그인 정보 안옴');
          }
        })
        .catch((error) => {
          console.log(error);
        });
    },
    getUserTimer() {
      let today = new Date();
      let day = date.dateFunc(today);

      this.$Axios
        .get('timer/today?datetime=' + day, {
          headers: {
            'jwt-auth-token': storage.getItem('jwt-auth-token'),
          },
        })
        .then((res) => {
          if (res.data.todayStudyTime > 0) {
            this.calCurrentTime(res.data.todayStudyTime); // sec -> min,hour 로 변환
            console.log('공부한 시간이 있어서 불러옴');
            console.log(res.data.todayStudyTime);
          } else {
            console.log('공부한 시간이 없어서 못 불러옴');
          }
        })
        .catch((error) => {
          console.log(error);
        });
    },
    pauseTimer() {
      console.log('pauseTimer');
      this.stopTimer();
    },
    resumeTimer() {
      console.log('resumeTimer');
      this.startTimer();
    },
    setDate() {
      let today = new Date();
      let year = today.getFullYear(); // 년도
      let month = today.getMonth() + 1; // 월
      let date = today.getDate(); // 날짜

      if (month < 10) {
        month = '0' + month;
      }
      if (date < 10) {
        date = '0' + date;
      }
      this.curruentDate = year + '-' + month + '-' + date;
    },
    getTodoList() {
      this.setDate();
      let today = new Date();
      let day = date.dateFunc(today);

      this.$Axios
        .get('todo?datetime=' + day, {
          headers: {
            'jwt-auth-token': storage.getItem('jwt-auth-token'),
          },
        })
        .then((res) => {
          if (Object.keys(res.data.todoList).length > 0) {
            this.todoList = res.data.todoList;
          } else {
            console.log('todoList 가져오기 실패');
          }
        })
        .catch((error) => {
          console.log(error);
        });
    },
    openTutorial() {
      this.dialog = true;
    },
    startTimer() {
      console.log('startTimer');
      // 타이머 시작 -> 카메라 on일때만 start 하는 걸로 변경필요. 새로고침하면 정보 사라지는데 어떡하지? vuex에 담아두기????
      this.changeStatusToStudy();
      // if (this.running) return; // 이미 타이머 돌아가고 잇는데 또 시작하라고할때

      // this.started = setInterval(() => this.runningTimer(), 1000); // 1s마다 타이머 함수가 실행되도록 함.

      // this.running = true; // 타이머가 실행되고 있음
    },
    stopTimer() {
      console.log('stopTimer');
      // this.running = false; // 타이머가 멈춤
      this.changeStatusToNotStudy();

      // clearInterval(this.started); // 반복 명령 종료
    },
    runningTimer() {
      if (this.$store.state.isStartCam) {
        // 타이머 시작
        var currentTime = new Date(); // 정각마다 공부한 시간을 저장하기 위해서
        this.UTCminute = currentTime.getUTCMinutes();
        this.UTCsecond = currentTime.getUTCSeconds();

        if (this.UTCminute === 0 && this.UTCsecond === 0) {
          // 정각일때마다 초기화 && DB에 누적된 초 저장
          console.log('정각이 되었습니다.');
          this.saveHourlyTime(); // 시간당 초를 저장함
        }

        this.accumulatedSec++; // 누적 초 저장
        this.sec++;
        if (this.sec >= 60) {
          this.min++;
          this.sec = 0;
          if (this.min >= 60) {
            this.hour++;
            this.min = 0;
          }
        }
        this.time =
          this.zeroPrefix(this.hour, 2) +
          ':' +
          this.zeroPrefix(this.min, 2) +
          ':' +
          this.zeroPrefix(this.sec, 2);
      }
    },
    zeroPrefix(num, digit) {
      var zero = '';
      for (var i = 0; i < digit; i++) {
        zero += '0';
      }
      return (zero + num).slice(-digit);
    },
    calCurrentTime(seconds) {
      this.hour = parseInt(seconds / 3600);
      this.min = parseInt((seconds % 3600) / 60);
      this.sec = seconds % 60;

      this.time =
        this.zeroPrefix(this.hour, 2) +
        ':' +
        this.zeroPrefix(this.min, 2) +
        ':' +
        this.zeroPrefix(this.sec, 2);
    },
    stopStudy() {
      this.$store.commit('setIsStartCam', false);
      this.$router.push('/');
      this.saveHourlyTime(); // 방해요소는 날짜 바뀔때랑 공부 종료할때 보내면 되고, 각 방해요소별로 보내야함.
      this.changeStatusToNotStudy();
      this.saveInterruption();
      // 비디오 끄기
      // 지금까지 공부한 시간 DB에 저장
    },
    changeStatusToStudy() {
      this.$Axios
        .create({
          headers: {
            'jwt-auth-token': storage.getItem('jwt-auth-token'),
          },
        })
        .put('user/start')
        .then((res) => {
          if (res.data.success) {
            // console.log('공부상태로 변경됨.');
            this.$store.commit('FETCHSTUDYING');
            this.runningTimer();
          } else {
            console.log('공부상태로 변경 실패');
          }
        })
        .catch((error) => {
          console.log(error);
        });
    },
    changeStatusToNotStudy() {
      this.$Axios
        .create({
          headers: {
            'jwt-auth-token': storage.getItem('jwt-auth-token'),
          },
        })
        .put('user/end')
        .then((res) => {
          if (res.data.success) {
            // console.log('비공부상태로 변경 성공');
            this.$store.commit('STOPSTUDYING');
          } else {
            console.log('비공부상태로 변경 실패');
          }
        })
        .catch((error) => {
          console.log(error);
        });
    },
    saveHourlyTime() {
      let today = new Date();
      let day = date.dateFunc(today);

      let params = new URLSearchParams();
      params.append('datetime', day); // 현재 시간에서 10초 이전의 시간을 보냄(알맞은 시간에) 저장되기 위해서)
      params.append('studyTime', this.accumulatedSec); // 누적된 초를 보냄

      // console.log(day);
      // console.log(this.accumulatedSec);

      this.$Axios
        .create({
          headers: {
            'jwt-auth-token': storage.getItem('jwt-auth-token'),
          },
        })
        .post('timer/study-time', params)
        .then((res) => {
          if (res.data.success) {
            this.accumulatedSec = 0; // 저장했으므로 다시 초기화함
          } else {
            console.log('공부시간 저장 실패!!!');
          }
        })
        .catch((error) => {
          console.log(error);
        });
    },
    saveInterruption() {
      let today = new Date();
      let day = date.dateFunc(today);

      var interruption = [
        `${this.$store.state.phoneTime}`,
        `${this.$store.state.awayTime}`,
        `${this.$store.state.talkTime}`,
        `${this.$store.state.sleepTime}`,
      ];

      for (var i = 0; i < interruption.length; i++) {
        if (interruption[i] == 0) continue; // 딴짓한게 없으면 보내지 않음

        let params = new URLSearchParams();
        params.append('action', i); // 딴짓의 종류(1:휴대폰, 2:자리비움, 3:잡담, 4:졸음)
        params.append('datetime', day);
        params.append('notStudyTime', interruption[i]); // 딴짓한 누적 시간

        this.$Axios
          .create({
            headers: {
              'jwt-auth-token': storage.getItem('jwt-auth-token'),
            },
          })
          .post('timer/not-study-time', params)
          .then((res) => {
            if (res.data.success) {
              // console.log('딴 짓한 시간 잘 저장됨');
            } else {
              console.log('딴 짓한 시간이 저장되지 않음');
            }
          })
          .catch((error) => {
            console.log(error);
          });
      }
      //방해 요인 시간을 0으로 만듬
      this.$store.commit('InitializeInterruption');
    },
    updateTodoList(value) {
      if (value) {
        // todoList 추가되었으면 다시 getTodoList()
        this.getTodoList();
      }
    },
  },
};
</script>

<style scoped>
.outbox {
  width: 100%;
  padding: 0;
}
.date {
  font-size: 25px;
  text-align: center;
  padding-top: 20px;
}
.todolist {
  font-size: 30px;
  padding-left: 20px;
  margin-top: 20px;
}
.user {
  display: inline-block;
  font-size: 18px;
  vertical-align: middle;
  padding-left: 7px;
  padding-top: 10px;
  /* float:left; */
}
.profile {
  padding-left: 10px;
}
.timer {
  vertical-align: middle;
  position: absolute;
  bottom: 5%;
  left: 33%;
}
.timerLogo {
  vertical-align: middle;
  position: absolute;
  top: 10px;
  left: 10px;
  max-width: 150px;
}
.container {
  max-width: 100vw !important;
}
.timerTutorial {
  vertical-align: middle;
  position: absolute;
  top: 10px;
  left: 67%;
  max-width: 150px;
}
#detectTable {
  position: absolute;
  top: 50px;
  left: 1%;
}
.startStudy {
  vertical-align: middle;
  position: absolute;
  top: 40%;
  left: 35%;
  font-size: 50px;
}
.time-bar {
  width: 250px;
  height: 70px;
  background-color: #2b243a;
  font-size: 3.2rem;
  color: white;
}
#tutorialTitle {
  font-weight: 500;
  padding-left: 10px;
}
#tutorialContent {
  font-weight: 500;
  font-size: 17px;
  padding: 20px;
}
.loadingCircular {
  position: absolute;
  left: 600px;
  top: 300px;
}
</style>
