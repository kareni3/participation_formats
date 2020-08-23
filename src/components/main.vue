<template>
  <div class="main" @keypress.enter="next()">
    <div class="container">
      <div class="content">
        <div class="question">
          <div v-if="page === 1">
            <div class="title">{{question.text}}</div>
            <div v-if="isFromats" class="answer_content formats">
              <div v-for="(el,i) in question.formats" :key="el">
                <input type="checkbox" v-model="usedFormatsCheckbox[i]" />
                <span>{{el}}</span>
              </div>
            </div>
          </div>
          <div v-else-if="page === 2">
            <div class="title">{{question.text}}</div>
            <div v-if="isFromats" class="answer_content formats">
              <div v-for="(el,i) in question.formats" :key="el">
                <input type="checkbox" v-model="valuelessFormatsCheckbox[i]" />
                <span>{{el}}</span>
              </div>
            </div>
          </div>
          <div v-else-if="page === 3">
            <div class="title">{{question.text}}</div>
            <div
              @click="showLevelConfidence=!showLevelConfidence"
              v-if="isFromats"
              class="label"
            >{{question.label}}</div>
            <div class="level_description">{{levelDescription[question.label]}}</div>
            <div class="confidence_container">
              <div
                v-show="showLevelConfidence"
                v-for="el in Object.entries(levelConfidence)"
                :key="el[0]"
              >{{el[0]}} - {{el[1]}}%</div>
            </div>
          </div>
          <div v-else-if="page === 4">
            <div class="title">{{question.text}}</div>
            <div v-if="isFromats" class="answer_content formats">
              <div v-for="(el,i) in question.levels" :key="el">
                <input type="checkbox" @input="selectLevelsCheckbox(i)" v-model="levelsCheckbox[i]" />
                <span>{{el}}</span>
              </div>
            </div>
          </div>
          <div v-else-if="page === 5">
            <div class="title">{{question.question}}</div>
            <div
              v-if="isFromats"
              class="answer_content"
              :class="question.type === 'number' ? 'inp' : 'formats'"
            >
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
            <div v-if="isFromats" :class="'answer_content formats'">
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
            <div class="recommendation_container">
              <div class="number">1</div>
              <div class="recommendation_wrapper" v-if="question.curLevelFormats.length">
                <div
                  class="recommendation_format_label"
                >You can try other participation formats of your level ({{participationLevel}}):</div>
                <div class="recommendation_format" v-for="el in question.curLevelFormats" :key="el">
                  <div>{{el}}</div>
                </div>
              </div>
              <div class="recommendation_wrapper" v-else>
                <div>None of participation formats for your level ({{participationLevel}}) we can recommend.</div>
                <img src="../assets/sorry.webp" />
              </div>
            </div>
            <div class="recommendation_container">
              <div class="number">2</div>
              <div class="recommendation_wrapper" v-if="question.uselessLevelFormats.length">
                <div
                  class="recommendation_format_label"
                >You said, that you have problems with some formats. Well, we only can recommend to you switch them to more useful ones. Please, avoid:</div>
                <div
                  class="recommendation_format recommendation_format__bad"
                  v-for="el in question.uselessLevelFormats"
                  :key="el"
                >
                  <div>{{el}}</div>
                </div>
              </div>
            </div>
            <div class="recommendation_container">
              <div class="number" v-if="!question.uselessLevelFormats.length">2</div>
              <div class="number" v-else>3</div>
              <div class="recommendation_wrapper" v-if="question.nextLevelFormats.length">
                <div
                  class="recommendation_format_label"
                >You can try participation formats of the next level ({{nextParticipationLevel}}):</div>
                <div
                  class="recommendation_format"
                  v-for="el in question.nextLevelFormats"
                  :key="el"
                >
                  <div>{{el}}</div>
                </div>
                <div
                  class="recommendation_confidence_label"
                >The confidence to move to the next level is:</div>
                <div>
                  <div
                    class="recommendation_confidence_value"
                    :class="confidenceColorClass"
                  >{{question.confidence}}%</div>
                  <div>
                    <div class="sprite" :style="`background-position: -${spriteShift * 800/5}px 0`"></div>
                  </div>
                </div>
                <div class="recommendation_confidence_message">
                  <div
                    class
                    v-if="question.confidence >= 80"
                  >You have excellent conditions to move to the next level</div>
                  <div
                    class
                    v-if="question.confidence >= 60 && question.confidence < 80"
                  >Move to the next level but do not forget about risks</div>
                  <div
                    class
                    v-if="question.confidence >= 45 && question.confidence < 60"
                  >Make the choice for yourself whether to move to the next level or not. The conditions for the transition are not ideal.</div>
                  <div
                    class
                    v-if="question.confidence >= 30 && question.confidence < 45"
                  >We do not recommend going to the next level. If you want to increase employee engagement, then, first, spend resources for their training, for their job satisfaction, save up funds to move to the next level, be more ready and open for new.</div>
                  <div
                    class
                    v-if="question.confidence >= 0 && question.confidence < 30"
                  >Stop, you should not go to the next level now. First, spend resources for employees training, for their job satisfaction, save up funds to move to the next level, be more ready and open for new.</div>
                </div>
              </div>
              <div class="recommendation_wrapper" v-else>
                <div>None of participation formats for the next level ({{nextParticipationLevel}}) we can recommend.</div>
                <img src="../assets/sorry.webp" />
              </div>
            </div>
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
      nextParticipationLevel: "",
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
      requirementNames: new Set(),
      levelConfidence: {},
      showLevelConfidence: false,
      levelDescription: {
        "No Involvement":
          "a management style where staff do not receive information or instruction from managers, and are not involved in operational or strategic decision-making.",
        "Information Participation":
          "Here, the employees are able to receive information and express their view relating to work and business in general. It is a management style where members/employees are provided with both written and verbal guidance by managers and/or governors, but are not invited or elected (individually or in groups) to contribute to operational or strategic decision-making.",
        "Consultative Participation":
          "The employees are consulted on matters related to their work and their welfare. The employees can only express their expectations and advice on related matters. However, the final decision rests with the management. It is a management style where members/employees (individually or in groups) have discussions about (pre-formed) management proposals, but are not invited or elected to participate in the formation of these proposals, or final decisions about their implementation.",
        "Associative Participation":
          "It is an extension of Consultative Participation where the management is under obligation to accept and implement the unanimous decisions of the employees. It is a management style where members/employees (individually or in groups) can participate in the development of ideas, and where the managers focus on coaching members/employees to develop their ideas into proposals, and support them during implementation. Managers retain some powers to screen-out weak proposals.",
        "Administrative Participation":
          "Here, employees have a greater share in the discharge of managerial functions. The employees are empowered to select the best from the alternative decisions for implementation. It is a management style where any member/employee (individually or in groups) can initiate discussions on operational or strategic issues, arrange and participate in meetings to develop proposals, and exercise both voice and voting power when decisions are made about implementation.",
        "Decisive Participation":
          "It is the highest level of participation where the decisions are jointly taken by the employees and the management on the matters related to production, welfare, introduction of change, etc. It is a management style where all of the decisions are taken together with managers and employees. ",
      },
    };
  },
  props: [
    "participationFormatsList",
    "questionsList",
    "clarifyFormatsQuestions",
  ],
  computed: {
    spriteShift() {
      let res = 0;
      if (!this.question && !this.question.confidence) return res;
      switch (true) {
        case this.question.confidence >= 80:
          res = 0;
          break;
        case this.question.confidence >= 60 && this.question.confidence < 80:
          res = 1;
          break;
        case this.question.confidence >= 45 && this.question.confidence < 60:
          res = 2;
          break;
        case this.question.confidence >= 30 && this.question.confidence < 45:
          res = 3;
          break;
        case this.question.confidence >= 0 && this.question.confidence < 30:
          res = 4;
          break;
      }
      return res;
    },
    confidenceColorClass() {
      let res = "";
      if (!this.question && !this.question.confidence) return res;
      switch (true) {
        case this.question.confidence >= 80:
          res = "recommendation_confidence_message__green";
          break;
        case this.question.confidence >= 60 && this.question.confidence < 80:
          res = "recommendation_confidence_message__lightgreen";
          break;
        case this.question.confidence >= 45 && this.question.confidence < 60:
          res = "recommendation_confidence_message__neutral";
          break;
        case this.question.confidence >= 30 && this.question.confidence < 45:
          res = "recommendation_confidence_message__lightred";
          break;
        case this.question.confidence >= 0 && this.question.confidence < 30:
          res = "recommendation_confidence_message__red";
          break;
      }
      return res;
    },
  },
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
      let isFirstQuestion = !v;
      let check = true;
      while (check) {
        let currentQuestion = null;
        if (isFirstQuestion) {
          currentQuestion = this.clarifyFormatsQuestions.find(
            (el) => el.isFirstQuestion
          );
        } else {
          if (this.question.type === "radiobuttons") {
            let answer = this.question.answers.find(
              (el) => el.label === this.answer
            );
            currentQuestion = this.clarifyFormatsQuestions.find(
              (e) =>
                e.questionNumber ===
                ((answer && answer.nextQuestionNumber) ||
                  this.question.questionNumber + 1)
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
          if (this.requirementNames.has(currentQuestion.questionName)) {
            check = false;
          } else {
            isFirstQuestion = false;
          }
        } else {
          this.questionNumber1 = 0;
          this.page++;
          check = false;
        }
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
            superRes[el[0]] = res1[el[0]] + res2[el[0]] || 0;
          });
          max = -Infinity;
          let sum = 0;
          let maxName = 0;
          this.levelConfidence = {};
          Object.entries(superRes).forEach((el) => {
            if (el[1] > max) {
              max = el[1];
              maxName = el[0];
            }
            sum += el[1];
          });
          Object.entries(superRes).forEach((el) => {
            this.levelConfidence[el[0]] = ((el[1] / sum) * 100).toFixed();
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
            let newname = Object.keys(obj[this.clarifyFormatsAnswers[name]])[0];
            let newobj = obj[this.clarifyFormatsAnswers[name]][newname];
            return rec(newobj, newname);
          }
        };
        Object.entries(this.participationFormatsList).forEach(
          (participationFormat) => {
            if (this.usedFormats.includes(participationFormat[0])) return;
            if (
              !participationFormat[1].levels.find(
                (e) =>
                  e.levelName === this.participationLevel ||
                  e.levelName === this.nextParticipationLevel
              )
            ) {
              return;
            }
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

        let str1 = [];
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
            str1.push(el[0]);
            arr1.push(el[0]);
          }
        });
        let str2 = [];
        this.valuelessFormats.forEach((el) => {
          str2.push(el);
        });
        let str3 = [];
        Object.entries(this.participationFormatsList).forEach((el) => {
          const participationLevel = el[1].levels.find(
            (e) => e.levelName === this.nextParticipationLevel
          );
          if (
            participationLevel &&
            !this.usedFormats.includes(el[0]) &&
            !arr1.includes(el[0]) &&
            !removedFormats.includes(el[0])
          ) {
            str3.push(el[0]);
          }
        });
        let confidence = 0;
        let weights = 0;
        Object.values(this.questionsList).forEach((el, i) => {
          const answer = el.answers.find(
            (e) => this.questionsListAnswers[i] === e.label
          );
          confidence +=
            (answer.answerWeight +
              (1 - answer.answerWeight) *
                el.questionLevelMultiplications[this.nextParticipationLevel]) *
            el.questionWeight; // M + ((1 - M) * N)
          weights += el.questionWeight;
        });
        confidence = (confidence / weights) * 100;
        this.question = {
          text: "Our recommendation to you",
          curLevelFormats: str1,
          uselessLevelFormats: str2,
          nextLevelFormats: str3,
          confidence: confidence.toFixed(),
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
        this.requirementNames = new Set();
        const index = this.allLevelsNames.indexOf(this.participationLevel);
        this.nextParticipationLevel = this.allLevelsNames[1 + index];
        Object.entries(this.participationFormatsList).forEach((el) => {
          if (this.usedFormats.includes(el[0])) return;
          if (
            !el[1].levels.find(
              (e) =>
                e.levelName === this.participationLevel ||
                e.levelName === this.nextParticipationLevel
            )
          ) {
            return;
          }
          this.requirementNames = new Set([
            ...this.requirementNames,
            ...Object.keys(el[1].requirements),
          ]);
          Object.values(el[1].requirements).forEach((el) => {
            if (!el.data) return;
            Object.values(el.data).forEach((e) => {
              if (typeof e === "object") {
                this.requirementNames = new Set([
                  ...this.requirementNames,
                  ...Object.keys(e),
                ]);
              }
            });
          });
        });
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
  overflow: hidden;
}
.content {
  min-height: calc(100% - 70px);
  margin-bottom: 70px;
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
  height: fit-content;
  margin-top: 24px;
  margin-bottom: 24px;
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
  background-color: #4169e1;
  margin-left: -24px;
  margin-right: -24px;
  padding: 24px;
  margin-top: -24px;
  color: white;
}
input {
  transform: scale(1.5);
}
.formats {
  width: 30%;
  margin: auto;
  width: fit-content;
}
.formats > div {
  margin-bottom: 12px;
  width: fit-content;
  display: flex;
}
.formats span {
  margin-left: 6px;
  display: inline-block;
}
.label {
  text-align: center;
  font-size: 3rem;
  padding-top: 24px;
  color: #4169e1;
  cursor: pointer;
}
.inp {
  width: 40%;
  margin: auto;
}
.inp input {
  transform: scale(1.5) translate(20%, 0);
  width: 70px;
}
.answer_content {
  padding: 25px 50px;
}
.confidence_container {
  color: #00000060;
  width: fit-content;
  margin: auto;
  padding-top: 12px;
}
.level_description {
  color: gray;
  width: 600px;
  text-align: justify;
  padding-top: 12px;
  margin: auto;
}
.recommendation_wrapper {
  text-align: center;
  width: 80%;
  margin: auto;
}
.recommendation_wrapper img {
  width: 320px;
  margin-top: 24px;
}
.recommendation_container {
  position: relative;
  margin-bottom: 84px;
}
.number {
  border: 1px solid #4169e1;
  line-height: 24px;
  width: 24px;
  position: absolute;
  text-align: center;
  padding: 40px;
  border-radius: 50%;
  color: #4169e1;
  left: -40px;
  top: -68px;
  font-size: 42px;
}
.recommendation_format {
  margin-bottom: 12px;
  text-align: center;
  color: #4169e1;
  font-size: 2rem;
}
.recommendation_format__bad {
  color: #ff6262;
}
.recommendation_format_label {
  text-align: center;
  margin-bottom: 24px;
}
.recommendation_confidence_message {
  text-align: center;
  padding-bottom: 24px;
}
.recommendation_confidence_value {
  text-align: center;
  text-align: center;
  color: #4169e1;
  font-size: 4rem;
  margin: 12px;
}
.recommendation_confidence_label {
  text-align: center;
  margin-top: 48px;
}
.recommendation_confidence_message__green {
  color: #00c300;
}
.recommendation_confidence_message__lightgreen {
  color: #4e884e;
}
.recommendation_confidence_message__neutral {
  color: #daa613;
}
.recommendation_confidence_message__lightred {
  color: #ff8484;
}
.recommendation_confidence_message__red {
  color: #e40000;
}
.sprite { 
    height: 153px;
    margin: 0 auto 12px auto;
    width: calc(800px / 5);
    background: url("../assets/mood.png");
}
</style>
