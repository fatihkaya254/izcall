<template lang="pug">
.main
    .plan(v-if="!lessons[callerID]")
      img(src="../assets/planv3.png") 
    .clock.mid
        p {{clock}}
    .student.mid
        p {{caller}}
    .lessons.mid(v-if="lessons[callerID]")
        .lesson(v-for="l in lessons[callerID]") 
          .lhour.mid {{l.appliedHour}}
          .lteacher.mid(v-if="l.teacher") {{l.teacher.name}} {{l.teacher.surname}}
          .lbranch.mid(v-if="l.branch") {{l.branch.branchName}}
          .lclass.mid(v-if="l.classroom") {{l.classroom.classroomName}}
          .lfloor.mid(v-if="l.classroom") {{l.classroom.floor}}. Kat
    .cardnum.mid
        p {{cardnum}}
</template>

<script>
import axios from "axios";
export default {
  name: "HelloWorld",
  props: {
    msg: String,
  },
  data() {
    return {
      card: "",
      cards: {},
      students: {},
      addStudent: "",
      cardnum: "",
      caller: "",
      callerID: "",
      clock: "",
      lessons: {},
      countdown: 0,
      daySettings: {
        0: {},
        1: {},
        2: {},
        3: {},
        4: {},
        5: {},
        6: {},
      },
      axiosHOST: "https://izders.com/api",
      axiosOPT: {
        headers: {
          common: {
            Authorization: "TBYEfaFGAsFoyiph9c4OmnFEWZQZrpSLT3RgGbJ93OTE7WdT",
          },
        },
      },
    };
  },
  methods: {
    startTime: function () {
      const today = new Date();
      let h = today.getHours();
      let m = today.getMinutes();
      let s = today.getSeconds();
      m = this.checkTime(m);
      s = this.checkTime(s);
      this.clock = h + ":" + m + ":" + s;
      setTimeout(this.startTime, 1000);
    },
    getStudent: async function () {
      var res = await axios.get(`${this.axiosHOST}/studentswn`, this.axiosOPT);
      this.students = res.data
    },
    checkTime: function (i) {
      if (i < 10) {
        i = "0" + i;
      } // add zero in front of numbers < 10
      return i;
    },
    getToday: async function () {
      var res = await axios.get(
        `${this.axiosHOST}/getTodayRecord`,
        this.axiosOPT
      );
      var todays = res.data.todays;
      todays.forEach((element) => {
        if (!this.lessons[element.student])
          this.$set(this.lessons, element.student, []);
        this.lessons[element.student].push(element);
      });
    },
    getRolls: async function () {
      var res = await axios.get(`${this.axiosHOST}/todaysRoll`, this.axiosOPT);
      var todays = res.data;
      todays.forEach((element) => {
        console.log(element);
      });
    },
    clear: function () {
      this.caller = "";
      this.callerID = "";
    },
    call: async function (card) {
      let res = await axios.post(
        `${this.axiosHOST}/rollCall`,
        { card: card },
        this.axiosOPT
      );
      console.log(res);
    },
    getCards: async function () {
      const res = await axios.get(`${this.axiosHOST}/getRC`, this.axiosOPT);
      for (const [key, val] of Object.entries(res.data)) {
          this.$set(this.cards, val.card, val);
        }
    },
    scan: function () {
      window.addEventListener("keydown", (e) => {
        if (this.cardnum.length >= 13) {
          this.cardnum = this.cardnum.substr(this.cardnum.length - 12);
        }
        if (e.key == "Enter" || e.key == "Escape") {
          let c = this.cards[this.cardnum];
          if (c) {
            this.call(this.cardnum);
            this.callerID = c.student;
            this.caller = this.students[c.student].name;
            var caller = this.students[c.student].name;
            let timeout = 20000;
            setTimeout(() => {
              if (this.caller == caller) {
                this.clear();
              }
            }, timeout);
          }
          this.cardnum = "";
        } else if (e.key.length == 1) {
          this.cardnum += e.key;
        }
      });
    },
  },
  mounted() {
    this.getCards();
    this.startTime();
    this.getStudent();
    this.scan();
    this.getToday();

    this.getRolls();
  },
};

</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->

<style lang="scss" scoped>
@use "../assets/colors.scss";
@import url("https://fonts.googleapis.com/css2?family=Roboto:ital,wght@0,100;0,300;0,400;0,500;0,700;0,900;1,100;1,300;1,400;1,500;1,700;1,900&display=swap");

.main {
  color: colors.$foreground-color;
  font-family: "Roboto", sans-serif;
  bottom: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  position: absolute;
  display: flex;
  flex-direction: column;
  background: radial-gradient(
    circle,
    colors.$gray6 0%,
    colors.$background-color 80%,
    colors.$background-color 100%
  );
  background-size: 300% 300%;
  background-color: red;
  animation: AnimateBG 20s ease infinite;
}
.mid {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}
.clock {
  height: 96px;
  font-size: 36px;
  font-weight: 100;
}
.student {
  height: 96px;
  font-size: 48px;
  font-weight: 300;
  color: colors.$blue;
}
.lessons {
  color: colors.$secondary-text;
}
.lesson {
  display: flex;
  height: 96px;
  width: 80%;
  border-bottom: 1px solid colors.$gray5;
}
.cardnum {
  width: 100%;
  height: 20px;
  font-size: 14px;
  font-weight: 100;
  position: absolute;
  bottom: 20px;
  left: 0;
}
.lhour {
  width: 20%;
  font-size: 36px;
  font-weight: 100;
}
.lteacher {
  width: 20%;
}
.lbranch {
  width: 20%;
}
.lclass {
  font-size: 36px;
  width: 20%;
}
.lfloor {
  font-size: 36px;
  font-weight: 100;
  width: 20%;
}
@keyframes AnimateBG {
  0% {
    background-position: 0% 24%;
  }
  50% {
    background-position: 100% 44%;
  }
  100% {
    background-position: 0% 24%;
  }
}
@keyframes AnimateOP {
  0% {
    opacity: 20%;
  }
  50% {
    opacity: 40%;
  }
  100% {
    opacity: 20%;
  }
}
.plan{
  position: absolute;
  top: 0;
  left: 0;
  height: 100vh;
  width: 100vw;
  display: flex;
  justify-content: center;
  align-items: center;
  img{
    animation: AnimateOP 12s ease infinite;
    opacity: 20%;
    width: 96vw;
  }
}
</style>
