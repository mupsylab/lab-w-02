<script>
import $ from "jquery";
import Session from "@/utils/session";
import Questionnaire from "@/utils/questionnaire";
import { initJsPsych } from 'jspsych';
import jsPsychHtmlKeyboardResponse from '@jspsych/plugin-html-keyboard-response';
import jsPsychHtmlButtonResponse from '@jspsych/plugin-html-button-response';
import jsPsychCallFunction from '@jspsych/plugin-call-function';
import jsPsychFullscreen from '@jspsych/plugin-fullscreen';
import jsPsychSurveyHtmlForm from '@jspsych/plugin-survey-html-form';

import Config from "@/config";
import Naodao from "@/utils/naodao";

import QuestionIdFirst from "@/components/Questionid1.vue";
import QuestionIdSecond from "@/components/Questionid2.vue";
import { h, createApp } from "vue";

const naodao = new Naodao();
const session = new Session({
  startTime: new Date().toLocaleDateString() + "-" + new Date().toLocaleTimeString(),
  experId: Config.experId,
  subjIdx: "002",
  finish: false
});
const jsPsych = initJsPsych({
  display_element: "exp",
  on_finish: () => {
    session.addInfo("endTime", new Date().toLocaleDateString() + "-" + new Date().toLocaleTimeString());
    session.changeInfo("finish", true);
    naodao.save();
  }
});
session.t = [
  "getData", () => {
    return jsPsych
      .data
      .get()
      .filter({ save: true })
      .addToAll(session.getAllInfo())
      .filterColumns(session.getInfoKeys().concat([
        "questionId", "answer"
      ]))
  }
];
naodao.getData = () => {
  return session.t["getData"]().csv();
}

const timeline = [{
  timeline: [{
    type: jsPsychHtmlKeyboardResponse,
    choices: "NO_KEYS",
    stimulus: () => {
      return "请用本地浏览器打开";
    }
  }],
  conditional_function: () => {
    var ua = navigator.userAgent.toLowerCase();
    if (ua.match(/micromessenger/i) == "micromessenger") {
      return true;
    } else {
      return false;
    }
  }
}, {
  type: jsPsychCallFunction,
  func: () => {
    jsPsych.pauseExperiment();
    let progress = {};
    let arr1 = jsPsych.utils.deepCopy(Config.assets);
    let arr2 = jsPsych.utils.deepCopy(Config.html);
    arr1.forEach((v, i) => {
      progress[v] = 0;
      new Promise(() => {
        let xhr = new XMLHttpRequest();
        xhr.open('GET', `./assets/media/${v}`, true);
        xhr.responseType = 'blob';
        xhr.onload = function (e) {
          if (this.status == 200) {
            var blob = this.response;
            session.media = [v, URL.createObjectURL(blob)];
          } else {
            alert("加载图片出错，位置于：" + v);
            location.reload();
          }
        };
        xhr.onprogress = (e) => {
          progress[v] = e.loaded / e.total;
        }
        xhr.ontimeout = function (event) {
          alert('请求超时！请检查你的网络！' + v);
          location.reload();
        }
        xhr.send();
      });
    });
    arr2.forEach((v, i) => {
      progress[v] = 0;
      new Promise(() => {
        let xhr = new XMLHttpRequest();
        xhr.open('GET', `./assets/external_html/${v}.html`, true);
        xhr.responseType = 'blob';
        xhr.onload = function (e) {
          if (this.status == 200) {
            this.response.text().then(e => {
              session.html = [v, e];
            });
          } else {
            alert("加载图片出错，位置于：" + v);
            location.reload();
          }
        };
        xhr.onprogress = (e) => {
          progress[v] = e.loaded / e.total;
        }
        xhr.ontimeout = function (event) {
          alert('请求超时！请检查你的网络！' + v);
          location.reload();
        }
        xhr.send();
      });
    });
    session.t = [
      "cca",
      setInterval(() => {
        let sum = 0;
        Object.keys(progress).forEach((v, i) => {
          sum += progress[v]
        });
        $("#jspsych-content").text(`
                正在加载实验资源，已加载：${parseInt(sum / (arr1.length + arr2.length) * 100).toString()}%
                `);
        if ((Object.keys(session.html).length + Object.keys(session.media).length) == (arr1.length + arr2.length)) {
          clearInterval(session.t["cca"]);
          jsPsych.finishTrial();
          jsPsych.resumeExperiment();
        }
      }, 500)
    ]
  }
}];

window.jsPsych = jsPsych;
window.session = session;

timeline.push({
  type: jsPsychSurveyHtmlForm,
  html: () => {
    return session.renderInfo([
      {
        type: "radio",
        name: "gender",
        required: true,
        choose: ["男", "女"],
        align: "col",
        prefix: "您的性别是"
      }
    ]);
  },
  button_label: "继续",
  on_finish: (data) => {
    Object.keys(data.response).forEach((v, i) => {
      session.addInfo(v, data.response[v]);
    });
  }
});

// if (Config.html.indexOf("instruction_part1") < 0) Config.html.push("instruction_part1");
// timeline.push({
//   type: jsPsychHtmlButtonResponse,
//   stimulus: () => {
//     return session.html["instruction_part1"]
//   },
//   choices: ["继续"]
// }, {
//   type: jsPsychCallFunction,
//   func: () => {
//     session.t = [
//       "nowQ",
//       0
//     ];
//   }
// }, {
//   timeline: [{
//     type: jsPsychSurveyHtmlForm,
//     html: () => {
//       return "<div id='Tbox'></div>";
//     },
//     button_label: "继续",
//     on_load: () => {
//       let now = Config.question1[session.t["nowQ"]];
//       createApp(h(QuestionIdFirst, {
//         id: session.t["nowQ"] + 1,
//         title: now.text,
//         choices: now.choices.split("-")
//       })).mount("#Tbox");
//       let cc = document.querySelector("#jspsych-survey-html-form-next");
//       cc.style.position = "relative";
//       cc.style.left = `calc(50% - ${cc.clientWidth / 2}px)`;
//     },
//     on_finish: (data) => {
//       Object.keys(data.response).forEach(v => {
//         jsPsych.data.write({
//           save: true,
//           questionId: v,
//           answer: data.response[v]
//         });
//       })
//       session.t["nowQ"]++;
//     }
//   }],
//   loop_function: () => {
//     return session.t["nowQ"] < Config.question1.length;
//   }
// });

if (Config.html.indexOf("instruction_part2") < 0) Config.html.push("instruction_part2");
timeline.push({
  type: jsPsychHtmlButtonResponse,
  stimulus: () => {
    return session.html["instruction_part2"]
  },
  choices: ["继续"]
}, {
  type: jsPsychCallFunction,
  func: () => {
    session.t = [
      "nowQ",
      0
    ];
    if (session.getInfo("gender") == "0") {
      session.t = [
        "q",
        Config.question2Man
      ]
    } else {
      session.t = [
        "q",
        Config.question2Woman
      ]
    }
  }
}, {
  timeline: [{
    type: jsPsychSurveyHtmlForm,
    html: () => {
      return "<div id='Tbox'></div>";
    },
    button_label: "继续",
    on_load: () => {
      let now = session.t["q"][session.t["nowQ"]];
      createApp(h(QuestionIdSecond, {
        id: session.t["nowQ"] + 1,
        title: now.text,
        choices: jsPsych.randomization.shuffle(now.choices)
      })).mount("#Tbox");
      let cc = document.querySelector("#jspsych-survey-html-form-next");
      cc.style.position = "relative";
      cc.style.left = `calc(50% - ${cc.clientWidth / 2}px)`;
    },
    on_finish: (data) => {
      Object.keys(data.response).forEach(v => {
        jsPsych.data.write({
          save: true,
          questionId: v,
          answer: data.response[v]
        });
      });
      session.t["nowQ"]++;
    }
  }],
  loop_function: () => {
    return session.t["nowQ"] < session.t["q"].length;
  }
});
timeline.push(
  {
    type: jsPsychFullscreen,
    fullscreen_mode: false
  }
);
export default {
  name: "App",
  mounted() {
    if (!Config.debug) session.startMonitor();

    jsPsych.run(timeline);
  }
}
</script>

<template>
  <div id="exp"></div>
</template>

<style scoped>
@import "jspsych/css/jspsych.css";
@import "@/assets/css/reset.css";
@import "@/assets/css/index.css";

#exp {
  width: 100%;
  height: 100%;
}
</style>
