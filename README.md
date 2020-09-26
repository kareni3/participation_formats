# Participation Formats

## Run Client:

### Preparation
- install vue and vue-cli
- install libraries:
```sh
cd ./PARTICIPATION_FORMATS
npm install
```

### Runing
- be sure that your server is running
- run:
```sh
cd ./PARTICIPATION_FORMATS
npm run serve
```

## How to change JSON files?

### questionsList.json

#### Description

This file store questions about the company, the company leaders, staff, etc. We ask them because we want to understand if the company ready to move to the next level of participation ot not.

The questions are mostly based on science literature (questionnaires, researchers, statistical data). Some of them are based on non-scientific posts, made by some managers. We believe them, because of two reasons: 1) we are not managers and do not know how it works in practice; 2) we base on several examples of such publications. For example, it is a question "How willing are you to experiment (take risks)?", that is not based on scientific literature. To add new questions here you should rely on such information.

#### Analyzing the Structure

questionWeight is a parameter, that gives the information if this question is important more or less than others.

```json
      "questionWeight": 0.7,
```

questionLevelMultiplications is a parameter, that gives question weights for all participation levels. First levels are not difficult to implement, so we add extra confidence to use their formats.

```json
"questionLevelMultiplications": {
        "Information Participation": 0.2,
        "Consultative Participation": 0.1,
        "Associative Participation": 0,
        "Administrative Participation": -0.05,
        "Decisive Participation": -0.1
      },
```

answerWeight is a parameter, that gives the information about how the answer influence the final confidence to implement next level of participation. The value is from 0 to 1. It is important to have edge values here.

```json
"answerWeight": 0.33
```

### clarifyFormatsQuestions.json

#### Description

This file store questions about the requirements. We ask them because we want to understand if the company is able to implement such and such format.

#### Analyzing the Structure

questionName is a parameter, that gives the identificator of the question. We will refer to it from "participationFormatsList.json"

```json
"questionName": "sizeOfCompany",
```

type is a parameter, that gives the type of the answers. It could be radiobuttons (list of checkboxes) or number (input field).

```json
"type": "radiobuttons",
```

nextQuestionNumber. If we want the next question to be next in turn, then we put here "null". Else, if the next question should be the other one (not in order), we put here the number of the next question.

```json
"nextQuestionNumber": 4
```

### participationFormatsList.json

#### Description

This file store the participation levels.

#### Analyzing the Structure

requirements. We put here the requirements from "clarifyFormatsQuestions.json". In "data" parameter we put answers on the question and one of 2 values: 1) true - the company meets the requirement 2) false - the opposite

```json
"requirements": {
        "canYouAllocateResourcesForNews": {
          "type": "condition",
          "data": {
            "Yes": true,
            "No": false
          }
        }
      },
```

weight. It is weight of format on different level. If level is not presentedm then its weight equals to 0. We had a rule that each first level of specific format has its own multiplication ("Information Participation" - 1, "Consultative Participation" - 1.1, "Associative Participation" - 1.2, "Administrative Participation" - 1.3, "Decisive Participation" - 1.4. For example, if the format has only levels "Associative Participation", "Administrative Participation", then we put 1.2 for "Associative Participation" and something for "Administrative Participation". For this "something" we have another rule - compare the format for this level and the first level of this format (here it is "Administrative Participation"). If the format is higher on the next level then we put here the weight more than the weight for the first level (here it could be 1.3), the opposite - we put less (here it could be 1).

For weights it is better to calculate them using Excel first, and only then put them here. In future, it is beter to use machine learning for this weights, cause sometimes it is difficult to understand what actually weight we need to put here or there.  

```json
"weight": 1
```

