<template>
  <div class="joinForm">
    <div>
      <p style="text-align:center;padding:20px;">
        (주)스윗미에서는 익명 사용자로 인한 피해를 방지하고자 [정보통신망
        이용촉진 및 정보보호 등에 관한 법률 제 23조의 2제 1항](주민등록번호의
        사용 제한)에 의거하여 주민등록번호 사용 및 수집을 중단하고 이메일 인증을
        통해 본인을 확인하고 있습니다.<br /><br />
      </p>
    </div>
    <v-container class="joinForm borderBox">
      <v-row style="padding-top:40px;">
        <v-col class="formLetter" cols="2">이메일</v-col>
        <v-col cols="3">
          <input-bar
            :isDisabled="isValid"
            :placeholder="'이메일 입력'"
            @pass-input="getEmailFront"
          ></input-bar>
        </v-col>
        <v-col cols="3" style="padding-left: 0;">
          <email-input
            :isDisabled="isValid"
            @pass-input="getEmailBack"
          ></email-input>
        </v-col>
        <v-col cols="3" align-self="center" align="center" @click="getdoubleck">
          <app-btn-middle
            :isDisabled="isDisabled"
            :btnColor="'#673fb4'"
            :btnName="'인증번호 받기'"
            :btnNameColor="'#ffffff'"
          ></app-btn-middle>
        </v-col>
      </v-row>
      <v-row style="padding-bottom:40px;">
        <v-col class="formLetter" cols="2">인증번호</v-col>
        <v-col cols="6">
          <input-bar
            :isDisabled="isValid"
            :placeholder="'4자리 숫자를 입력 해주세요.'"
            @pass-input="getMyAuthNum"
          ></input-bar>
        </v-col>
        <v-col
          cols="3"
          align-self="center"
          align="center"
          @click="compareAuthNum"
        >
          <app-btn-middle
            :isDisabled="isDisabled2"
            :btnColor="'#673fb4'"
            :btnName="'확인'"
            :btnNameColor="'#ffffff'"
          ></app-btn-middle>
        </v-col>
      </v-row>
    </v-container>
    <div style="text-align:center;">
      <div class="oneBtn" @click="$router.push('/join')">
        <app-btn-middle
          :btnColor="'#FAFAFA'"
          :btnName="'이전'"
          :btnNameColor="'#424242'"
        ></app-btn-middle>
      </div>
      <div class="oneBtn" @click="goRouting">
        <app-btn-middle
          :isDisabled="!isValid"
          :btnColor="'#424242'"
          :btnName="'다음'"
          :btnNameColor="'white'"
        ></app-btn-middle>
      </div>
    </div>
  </div>
</template>

<script>
import "./user.css";
import InputBar from "@/components/common/InputBar.vue";
import EmailInput from "@/components/common/EmailInput.vue";
import AppBtnMiddle from "@/components/common/AppBtnMiddle.vue";
import axios from "axios";

export default {
  components: {
    "input-bar": InputBar,
    "email-input": EmailInput,
    "app-btn-middle": AppBtnMiddle,
  },
  created: function() {
    this.$emit("move-step", 2);
  },
  data: function() {
    return {
      emailFront: "",
      emailBack: "",
      myAuthNum: 1234567890,
      realAuthNum: 0,
      isValid: false,
      // isDisabled 쓰는 이유... 버튼이 component라서 @click을 바로 못 주니까 임시방편으로 플래그 만든거임 ㅜ
      isDisabled2: true,
    };
  },
  computed: {
    isDisabled() {
      if (this.emailFront != "" && this.emailBack != "") {
        return false;
      } else {
        return true;
      }
    },
  },
  methods: {
    goRouting() {
      if (this.isValid) {
        this.$emit("move-forward");
        this.$router.push(
          "/join/join-create?emailF=" +
            this.emailFront +
            "&emailB=" +
            this.emailBack
        );
      }
    },
    getEmailFront(value) {
      this.emailFront = value;
    },
    getEmailBack(value) {
      this.emailBack = value;
    },
    getdoubleck() {
      if (!this.isDisabled) {
        axios
          .get("user/id?userId=" + this.emailFront + "@" + this.emailBack)
          .then((response) => {
            if (response.data.isPresent) {
              alert("중복된 이메일 입니다. 다른 이메일 주소를 입력해주세요.");
            } else {
              this.getAuthNum();
              alert("입력하신 이메일로 인증번호를 발송했습니다.");
            }
          });
      }
    },
    getAuthNum() {
      this.$Axios
        .get("user/email?userEmail=" + this.emailFront + "@" + this.emailBack)
        .then((response) => {
          if (response.data.success) {
            this.realAuthNum = response.data.validNum;
            this.isDisabled2 = false;
          }
        })
        .catch((error) => {
          console.log(error);
        });
    },
    getMyAuthNum(value) {
      this.myAuthNum = value;
    },
    compareAuthNum() {
      if (!this.isDisabled && this.myAuthNum == this.realAuthNum) {
        alert("이메일 인증이 완료되었습니다.");
        this.isDisabled = true;
        this.isValid = true;
      } else if (this.myAuthNum != this.realAuthNum) {
        alert("인증번호를 정확히 입력해주세요.");
        this.isValid = false;
      }
    },
  },
};
</script>

<style></style>
