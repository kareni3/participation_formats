<template>
  <div class="main">
    <div class="container">
      <div class="content">
        <div class="question">
          <div v-if="page === 1">
            <div class="title">{{question.text}}</div>
            <div v-if="isFromats" class="formats">
              <div v-for="(el,i) in question.formats" :key="el">
                <input type="checkbox" v-model="usedFormatsCheckbox[i]" />
                <span>{{el}}</span>
              </div>
            </div>
          </div>
          <div v-else-if="page === 2">
            <div class="title">{{question.text}}</div>
            <div v-if="isFromats" class="formats">
              <div v-for="(el,i) in question.formats" :key="el">
                <input type="checkbox" v-model="valuelessFormatsCheckbox[i]" />
                <span>{{el}}</span>
              </div>
            </div>
          </div>
          <div v-else-if="page === 3">
            <div class="title">{{question.text}}</div>
            <div v-if="isFromats" class="label">{{question.label}}</div>
          </div>
          <div v-else-if="page === 4">
            <div class="title">{{question.text}}</div>
            <div v-if="isFromats" class="formats">
              <div v-for="(el,i) in question.levels" :key="el">
                <input type="checkbox" @input="selectLevelsCheckbox(i)" v-model="levelsCheckbox[i]" />
                <span>{{el}}</span>
              </div>
            </div>
          </div>
          <div v-else-if="page === 5">
            <div class="title">{{question.question}}</div>
            <div v-if="isFromats" :class="question.type === 'number' ? 'inp' : 'formats'">
              <div v-for="(el,i) in question.answers" :key="i">
                <div v-if="question.type === 'radiobuttons'">
                  <input
                    type="checkbox"
                    @input="selectAnswersCheckbox(i)"
                    v-model="answersCheckbox[i]"
                  />
                  <span>{{el.label}}</span>
                </div>
                <div v-else-if="question.type === 'number'">
                  <span>{{el.label}}</span>
                  <input type="number" v-model="answer" />
                </div>
              </div>
            </div>
          </div>
          <div v-else-if="page === 6">
            <div class="title">{{question.question}}</div>
            <div v-if="isFromats" :class="'formats'">
              <div v-for="(el,i) in question.answers" :key="i">
                <div>
                  <input
                    type="checkbox"
                    @input="selectAnswersCheckbox(i)"
                    v-model="answersCheckbox[i]"
                  />
                  <span>{{el.label}}</span>
                </div>
              </div>
            </div>
          </div>
          <div v-else-if="page === 7">
            <div class="title">{{question.text}}</div>
            <div>{{question.label1}}</div>
            <div>{{question.label2}}</div>
            <div>{{question.label3}}</div>
          </div>
        </div>
      </div>
      <div class="action_container">
        <div class="action">
          <div class="btn back">{{backName}}</div>
          <div v-if="scipName" class="btn scip" @click="scip">{{scipName}}</div>
          <div v-if="nextName" class="btn next" @click="next">{{nextName}}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "Main",
  data() {
    return {
      question: null,
      isFromats: false,
      participationLevel: "",
      usedFormatsCheckbox: [],
      usedFormats: [],
      UsedFormats: [],
      valuelessFormatsCheckbox: [],
      valuelessFormats: [],
      ValuelessFormats: [],
      levelsCheckbox: [],
      page: 0,
      nextName: "NEXT",
      backName: "BACK",
      scipName: "SCIP",
      clarifyFormatsAnswers: {},
      questionsListAnswers: [],
      questionNumber: null,
      questionNumber1: null,
      answer: null,
      answersCheckbox: [],
      allLevelsNames: [],
    };
  },
  props: [
    "participationFormatsList",
    "questionsList",
    "clarifyFormatsQuestions",
  ],
  watch: {
    usedFormatsCheckbox(v) {
      this.usedFormats = Object.keys(this.participationFormatsList).filter(
        (el, i) => {
          return v[i];
        }
      );
    },
    valuelessFormatsCheckbox(v) {
      this.valuelessFormats = this.usedFormats.filter((el, i) => {
        return v[i];
      });
    },
    questionNumber(v) {
      if (!v && v !== 0) return;
      let currentQuestion = null;
      if (v === 0) {
        currentQuestion = this.clarifyFormatsQuestions.find(
          (el) => el.isFirstQuestion
        );
      } else {
        if (this.question.type === "radiobuttons") {
          currentQuestion = this.clarifyFormatsQuestions.find(
            (e) =>
              e.questionNumber ===
              (this.question.answers.find((el) => el.label === this.answer)
                .nextQuestionNumber || this.question.questionNumber + 1)
          );
        } else if (this.question.type === "number") {
          currentQuestion = this.clarifyFormatsQuestions.find(
            (e) => e.questionNumber === this.question.questionNumber + 1
          );
        }
        this.answer = null;
        this.answersCheckbox = [];
      }
      if (currentQuestion) {
        this.question = currentQuestion;
      } else {
        this.questionNumber1 = 0;
        this.page++;
      }
    },
    questionNumber1(v) {
      if (!v && v !== 0) return;
      let currentQuestion = null;
      if (v === 0) {
        currentQuestion = this.questionsList.find(
          (el) => el.questionNumber === 1
        );
      } else {
        currentQuestion = this.questionsList.find(
          (e) => e.questionNumber === this.question.questionNumber + 1
        );
        this.answer = null;
        this.answersCheckbox = [];
      }
      if (currentQuestion) {
        this.question = currentQuestion;
      } else {
        this.page++;
      }
    },
    page() {
      if (this.page === 2) {
        if (this.usedFormats.length === 0) this.page++;
        this.question = {
          text:
            "Select participation formats that you think are not working well (valueless, too expensive, etc.)",
          formats: this.usedFormats,
        };
        this.scipName = "";
      }
      if (this.page === 3) {
        this.UsedFormats = Object.entries(this.participationFormatsList).filter(
          (el) => {
            return this.usedFormats.includes(el[0]);
          }
        );
        this.ValuelessFormats = Object.entries(
          this.participationFormatsList
        ).filter((el) => {
          return this.valuelessFormats.includes(el[0]);
        });
        if (this.participationLevel) {
          this.question = {
            text: "Your current level of participation is",
            label: this.participationLevel,
          };
        } else if (this.UsedFormats.length > this.ValuelessFormats.length) {
          const levelsWeightsSum = {};
          Object.values(this.participationFormatsList).forEach((el) => {
            el.levels.forEach((el1) => {
              if (!levelsWeightsSum[el1.levelName])
                levelsWeightsSum[el1.levelName] = 0;
              levelsWeightsSum[el1.levelName] += el1.weight;
            });
          });
          const usedLevelsWeightsSum = {};
          this.UsedFormats.forEach((el) => {
            el[1].levels.forEach((el1) => {
              if (!usedLevelsWeightsSum[el1.levelName])
                usedLevelsWeightsSum[el1.levelName] = 0;
              if (!this.ValuelessFormats.find((e) => e[0] === el[0])) {
                usedLevelsWeightsSum[el1.levelName] += el1.weight;
              }
            });
          });
          let max = -Infinity;
          Object.entries(usedLevelsWeightsSum).forEach((el) => {
            if (el[1] > max) max = el[1];
          });
          const res1 = {};
          const res2 = {};
          const superRes = {};
          Object.entries(levelsWeightsSum).forEach((el) => {
            res1[el[0]] = usedLevelsWeightsSum[el[0]] / el[1];
            res2[el[0]] = usedLevelsWeightsSum[el[0]] / max;
            superRes[el[0]] = res1[el[0]] + res2[el[0]];
          });
          max = -Infinity;
          let maxName = 0;
          Object.entries(superRes).forEach((el) => {
            if (el[1] > max) {
              max = el[1];
              maxName = el[0];
            }
          });
          this.question = {
            text: "Your current level of participation is",
            label: maxName,
          };
          this.participationLevel = maxName;
        } else {
          this.question = {
            text: "Your current level of participation is",
            label: "No Involvement",
          };
          this.participationLevel = "No Involvement";
        }
        this.scipName = "DISAGREE";
        this.nextName = "AGREE";
      }
      if (this.page === 4) {
        this.scipName = "";
        const levels = [];
        levels.push("No Involvement");
        Object.values(this.participationFormatsList).forEach((el) => {
          el.levels.forEach((el1) => {
            if (!levels.includes(el1.levelName)) levels.push(el1.levelName);
          });
        });
        this.question = {
          text: "Select your current level of participation",
          levels: levels,
        };
      }
      if (this.page === 7) {
        const removedFormats = [];
        const rec = (obj, name) => {
          if (typeof obj[this.clarifyFormatsAnswers[name]] === "boolean") {
            return obj[this.clarifyFormatsAnswers[name]];
          } else {
            let newname = Object.keys(obj[this.clarifyFormatsAnswers[name]])[0]
            let newobj = obj[this.clarifyFormatsAnswers[name]][newname]
            return rec(newobj, newname);
          }
        };
        Object.entries(this.participationFormatsList).forEach(
          (participationFormat) => {
            Object.entries(participationFormat[1].requirements).forEach(
              (requirement) => {
                if (requirement[1].type === "range") {
                  let min = requirement[1].data[0];
                  let max = requirement[1].data[1];
                  if (max === "Infinity") max = Infinity;
                  if (
                    this.clarifyFormatsAnswers[requirement[0]] < min ||
                    this.clarifyFormatsAnswers[requirement[0]] > max
                  ) {
                    removedFormats.push(participationFormat[0]);
                  }
                } else if (requirement[1].type === "condition") {
                  const res = rec(requirement[1].data, requirement[0]);
                  if (!res) removedFormats.push(participationFormat[0]);
                }
              }
            );
          }
        );

        let str1 = "";
        let arr1 = [];
        Object.entries(this.participationFormatsList).forEach((el) => {
          const participationLevel = el[1].levels.find(
            (e) => e.levelName === this.participationLevel
          );
          if (
            participationLevel &&
            !this.usedFormats.includes(el[0]) &&
            !removedFormats.includes(el[0])
          ) {
            str1 += `${el[0]}, `;
            arr1.push(el[0]);
          }
        });
        str1 = str1.slice(0, -2);
        let str2 = "";
        this.valuelessFormats.forEach((el) => {
          str2 += `${el}, `;
        });
        str2 = str2.slice(0, -2);
        let str3 = "";
        let index = this.allLevelsNames.indexOf(this.participationLevel);
        const nextLevel = this.allLevelsNames[1 + index];
        Object.entries(this.participationFormatsList).forEach((el) => {
          const participationLevel = el[1].levels.find(
            (e) => e.levelName === nextLevel
          );
          if (
            participationLevel &&
            !this.usedFormats.includes(el[0]) &&
            !arr1.includes(el[0]) &&
            !removedFormats.includes(el[0])
          ) {
            str3 += `${el[0]}, `;
          }
        });
        str3 = str3.slice(0, -2);
        let confidence = 0;
        let weights = 0;
        Object.values(this.questionsList).forEach((el, i) => {
          confidence +=
            el.answers.find((e) => this.questionsListAnswers[i] === e.label)
              .answerWeight * el.questionWeight;
          weights += el.questionWeight;
        });
        confidence = (confidence / weights) * 100;
        this.question = {
          text: "Our recommendation to you",
          label1: "Try these formats (your level): " + str1,
          label2: "Don't use these formats: " + str2,
          label3:
            `Confidence - ${confidence.toFixed()}% -> Try these formats (next level): ` +
            str3,
        };
      }
    },
  },
  mounted() {
    this.question = {
      text: "Select participation formats that you use in your company.",
      formats: Object.keys(this.participationFormatsList),
    };
    this.isFromats = true;
    this.page = 1;
    this.scipName = "SELECT PARTICIPATION LEVEL";
    this.allLevelsNames.push("No Involvement");
    Object.entries(this.participationFormatsList).forEach((el) => {
      el[1].levels.forEach((e) => {
        if (!this.allLevelsNames.includes(e.levelName))
          this.allLevelsNames.push(e.levelName);
      });
    });
  },
  methods: {
    next() {
      if (this.page === 3) {
        this.page = 5;
        this.questionNumber = 0;
        this.scipName = "";
        this.nextName = "NEXT";
      } else if (this.page === 4) {
        this.page = 3;
      } else if (this.page === 5) {
        this.clarifyFormatsAnswers[this.question.questionName] = this.answer;
        this.questionNumber++;
      } else if (this.page === 6) {
        this.questionsListAnswers.push(this.answer);
        this.questionNumber1++;
      } else {
        this.page++;
      }
    },
    scip() {
      if (this.page === 1 || this.page === 3) {
        this.page = 4;
      }
    },
    selectLevelsCheckbox(i) {
      this.levelsCheckbox = [];
      this.levelsCheckbox[i] = true;
      this.participationLevel = this.question.levels[i];
    },
    selectAnswersCheckbox(i) {
      this.answersCheckbox = [];
      this.answersCheckbox[i] = true;
      this.answer = this.question.answers[i].label;
    },
  },
};
</script>


<style scoped>
.main {
  padding: 24px;
  font-size: 1.2rem;
}
.container {
  /* position:inherit;
  height: calc(100% - 48px);
  width: calc(100% - 48px); */
}
.content {
  min-height: 90%;
  margin-bottom: 10%;
}
.action {
  display: flex;
  align-items: center;
  flex-direction: row;
  justify-content: space-between;
  width: 90%;
  margin: auto;
}
.action_container {
  height: 10%;
  position: absolute;
  bottom: 0;
  width: calc(100% - 48px);
}
.btn {
  cursor: pointer;
  padding: 12px 24px;
  background-color: #4169e1;
  color: white;
  font-weight: 600;
  white-space: nowrap;
  text-overflow: ellipsis;
  overflow: hidden;
}
.scip {
  background-color: #fa8d8d;
}
.back {
  background-color: #c4c4c4;
}
.next {
}
.title {
  font-size: 2rem;
  text-align: center;
  margin-bottom: 48px;
}
input {
  transform: scale(1.5);
}
.formats {
  width: 30%;
  margin: auto;
}
.formats > div {
  margin-bottom: 6px;
  display: flex;
}
.formats span {
  margin-left: 6px;
  display: inline-block;
}
.label {
  text-align: center;
  font-size: 3rem;
  padding-top: 48px;
  color: #4169e1;
}
.inp {
  width: 40%;
  margin: auto;
}
.inp input {
  transform: scale(1.5) translate(20%, 0);
  width: 70px;
}
</style>
