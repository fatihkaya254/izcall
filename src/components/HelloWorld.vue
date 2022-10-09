<template lang="pug">
.main
    .plan(v-if="callerID == ''")
      img(src="../assets/planv2.png") 
    .plan2(v-if="callerID == ''")
      img(src="../assets/planv3.png") 
    .icons
      .icon.green(v-if="isOnline")
      .icon.red(v-if="!isOnline")
      .icon.orange(v-if="axiosError")
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
      validKeys:["0","1","2","3","4","5","6","7","8","9"], 
      axiosError: false,
      callList: [],
      isOnline: navigator.onLine,
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
      console.log('getStudent()');
      try {
        this.axiosError = false;
        var res = await axios.get(
          `${this.axiosHOST}/studentswn`,
          this.axiosOPT
        );
        this.students = res.data;
        localStorage.setItem("students", JSON.stringify(res.data));
      } catch (error) {
        this.axiosError = true;
        var students = localStorage.getItem("students");
        if (students) this.students = JSON.parse(students);
        console.log(error);
      }
    },
    checkTime: function (i) {
      if (i < 10) {
        i = "0" + i;
      } // add zero in front of numbers < 10
      return i;
    },
    getToday: async function () {
      console.log('getToday()');

      try {
        this.axiosError = false;
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
        localStorage.setItem("todays", JSON.stringify(res.data.todays));
      } catch (error) {
        this.axiosError = true;
        var today = [];
        if (localStorage.getItem("todays"))
          today = JSON.parse(localStorage.getItem("todays"));
        today.forEach((element) => {
          if (!this.lessons[element.student])
            this.$set(this.lessons, element.student, []);
          this.lessons[element.student].push(element);
        });
        console.log(error);
      }
    },
    getRolls: async function () {
      console.log('getRolls()');
      try {
        this.axiosError = false;
        var res = await axios.get(
          `${this.axiosHOST}/todaysRoll`,
          this.axiosOPT
        );
        var todays = res.data;
        todays.forEach((element) => {
          console.log(element);
        });
      } catch (error) {
        this.axiosError = true;
        console.log(error);
      }
    },
    clear: function () {
      this.caller = "";
      this.callerID = "";
    },
    call: async function (card) {
      try {
        this.axiosError = false;
        let res = await axios.post(
          `${this.axiosHOST}/rollCall`,
          { card: card },
          this.axiosOPT
        );
        if (res.data.card) {
          var index = this.callList.indexOf(card);
          if (index !== -1) {
            this.callList.splice(index, 1);
          }
        }
        localStorage.setItem('callList', this.callList)
      } catch (error) {
        this.axiosError = true;
        console.log(error);
      }
    },
    getCards: async function () {
      console.log('getCards()');
      try {
        this.axiosError = false;
        const res = await axios.get(`${this.axiosHOST}/getRC`, this.axiosOPT);
        localStorage.setItem("cards", JSON.stringify(res.data));
        for (const [key, val] of Object.entries(res.data)) {
          this.$set(this.cards, val.card, val);
        }
      } catch (error) {
        this.axiosError = true;
        let cardcard = localStorage.getItem("cards");
        if (cardcard) {
          let cards = JSON.parse(cardcard);
          for (const [key, val] of Object.entries(cards)) {
            this.$set(this.cards, val.card, val);
          }
        }
        console.log(error);
      }
    },
    getDateToday: function () {
      var today = new Date();
      var dd = String(today.getDate()).padStart(2, "0");
      var mm = String(today.getMonth() + 1).padStart(2, "0"); //January is 0!
      var yyyy = today.getFullYear();
      return dd + "." + mm + "." + yyyy;
    },
    localCallList: function () {
      var check = localStorage.getItem("date");
      var date = this.getDateToday();
      if (date != check) {
        console.log('not');
        localStorage.setItem("date", date);
        localStorage.removeItem("callList");
        localStorage.removeItem("todays");
        this.getCards();
        this.getStudent();
        this.getToday();
      }
      var list = localStorage.getItem("callList");
      if (list) this.callList = list.split(",");
    },
    scan: function () {
      window.addEventListener("keydown", (e) => {
        if (this.cardnum.length >= 11) {
          this.cardnum = this.cardnum.substr(this.cardnum.length - 10);
        }
        if (e.key == "Enter" || e.key == "Escape") {
          if(this.cardnum == 1234) localStorage.clear()
          this.localCallList();
          this.isOnline = navigator.onLine;
          let c = this.cards[this.cardnum];
          if (c) {
            if (!this.callList.includes(this.cardnum))
              this.callList.push(this.cardnum);
            localStorage.setItem("callList", this.callList);
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
          this.callList.forEach((e) => {
            this.call(e);
          });
          this.cardnum = "";
        } else if (this.validKeys.includes(e.key)) {
          this.cardnum += e.key;
        }
      });
    },
  },
  mounted() {
    this.localCallList();
    this.startTime();
    this.getStudent();
    this.scan();
    this.getToday();
    this.getCards();
    //this.getRolls();
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
  overflow: hidden;
  background: radial-gradient(
    circle,
    colors.$gray6 0%,
    colors.$background-color 80%,
    colors.$background-color 100%
  );
  background-size: 300% 300%;
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
.plan {
  position: absolute;
  top: 0;
  left: 0;
  height: 67vh;
  width: 100vw;
  display: flex;
  justify-content: center;
  align-items: flex-end;
  img {
    height: 60vh;
  }
}
.plan2 {
  position: absolute;
  bottom: 0;
  left: 0;
  height: 30vh;
  width: 100vw;
  display: flex;
  justify-content: center;
  align-items: flex-start;
  img {
    height: 25vh;
  }
}
.icon {
  background-color: colors.$gray6;
  height: 12px;
  width: 12px;
  border-radius: 50%;
}
.green {
  background-color: colors.$green;
}
.red {
  background-color: colors.$red;
}
.orange {
  background-color: colors.$orange;
}
.icons {
  height: 24px;
  display: flex;
  justify-content: space-evenly;
  align-items: center;
  width: 72px;
  position: absolute;
  top: 12px;
  left: 12px;
}
</style>
