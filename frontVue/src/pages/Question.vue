<template>
    <v-container class="grey lighten-5">
        <!-- 一時保管 -->
        <!-- <button @click="plus">+1</button>
            <h1>{{ this.$store.state.count }}</h1> -->

        <v-row>
            <v-col cols="1"></v-col>
            <v-col>
                <h1>問題を解け</h1>
            </v-col>
            <v-col cols="1"></v-col>
        </v-row>

        <v-row>
            <v-col></v-col>
        </v-row>

        <v-row>
            <v-col cols="1"></v-col>
            <v-col>
                <div>
                    <h2>{{this.$store.state.userAnswer.length + 1}}.{{ this.displayQuestion }}</h2>
                </div>
            </v-col>
            <v-col></v-col>
        </v-row>

        <v-row>
            <v-col></v-col>
        </v-row>

        <v-row>
            <v-col cols="1"></v-col>
            <v-col>
                <MyCanvas ref="reset"></MyCanvas>
            </v-col>
            <v-col v-if="this.currentAnswer.length == 2">
                <MyCanvas></MyCanvas>
            </v-col>

            <v-col cols="7">

                <v-btn

                        color="warning"
                        @click="predictKanji"
                        class="ma-2 font-weight-black"
                        depressed
                        small
                        large
                >
                    回答する
                </v-btn>


                <v-col>
                    <v-row
                    ><span class="title font-weight-bold">{{ moji[0] }}</span></v-row
                    >
                    <v-row>
                        <v-col></v-col>
                    </v-row>
                    <v-row>
                        <div class="display-4">{{ kanji[number] }}</div>
                        <v-col cols="1"></v-col>
                    </v-row>
                    <v-col>
                        <div class="title font-weight-bold">{{ moji[2] }}</div>
                    </v-col>
                    <v-row>
                        <!--<v-col cols="1"></v-col>-->

                        <div v-if="kanji">
                            <span v-for="n in 5">
                                <v-btn
                                        v-if="$store.state.userAnswer.length < 3"
                                        min-height="100px"
                                        min-width="100px"
                                        tile
                                        outlined
                                        color="burakku"
                                        @click="choiceKanji(n); pushAnswer "


                                ><span class="display-3 font-weight-black">{{
                                    kanji[n - 1]
                                    }}</span></v-btn>
                            </span>


                            <!--<v-btn-->
                            <!--v-if="$store.state.userAnswer.length < 2"-->
                            <!--@click="pushAnswer"-->
                            <!--depressed-->
                            <!--color="#FF8100"-->
                            <!--valign="bottom"-->
                            <!--&gt;次の問題-->
                            <!--</v-btn-->
                            <v-btn
                                    @click="createResult"
                                    v-if="$store.state.userAnswer.length == 3"
                                    depressed
                                    color="#FF8100"
                                    valign="bottom"
                                    to="Result"
                                    x-large
                                    dark
                            >結果を見る
                            </v-btn>


                        </div>
                        <span v-if="kanji">
                        <v-row>

                            <v-col></v-col>

                        </v-row>
                        </span>
                    </v-row>
                </v-col>

            </v-col>

        </v-row>
        </v-col>
        <v-col>
        </v-col>
        </v-row>
        <v-row>

        </v-row>
        <v-row>
            <v-col></v-col>
        </v-row>
        <v-row>
            <v-col cols="1"></v-col>
            <v-col cols="2"></v-col>
        </v-row>
    </v-container>
</template>

<script>
  import MyCanvas from "../components/CanbasL";
  import QuestionSentence from "@/components/QuestionSentence.vue";
  import axios from "axios";
  import {log} from "util";

  export default {
    name: "Question",

    data() {
      return {
        query: "",
        msg: "",
        qInput: ["", "", "", "", "", "", "", "", "", ""],
        qOutput: ["", "", "", "", "", "", "", "", "", ""],
        userAnswer: [],
        tmp: "",
        object: "",
        kanji: "",
        onyomi: "",
        kunyomi: "",
        kakusu: "",
        number: "0",
        moji: "",
        i: "0",
      };
    },

    components: {
      QuestionSentence,
      MyCanvas
    },

    created() {
      this.getQuestionAndAnswer();
    },

    computed: {
      displayQuestion() {
        return this.$store.state.question[this.$store.state.userAnswer.length];
      },
      currentAnswer() {
        return new String(this.$store.state.answer[this.userAnswer.length]);
      }
    },

    methods: {
      predictKanji() {

        this.$store.commit("pushAnswer");

        const POST_URL = process.env.VUE_APP_URL_BASE + "kanji_search.py";
        let params = new URLSearchParams();
        params.append("object", JSON.stringify(this.$store.state.canvasArray[this.$store.state.canvasArray.length - 1]));
        //ここにURL指定。

        axios
          .post(POST_URL, params)
          .then(response => {
            this.kanji = response.data.kanji;
            this.onyomi = response.data.onyomi;
            this.kunyomi = response.data.kunyomi;
            this.kakusu = response.data.kakusu;
            this.moji = ["あなたの書いた字はこちらですか、", "選んでください。 : ", "選んでください。", "画数　 : ", "あなたの書いた漢字はこちらですか？"];
          })
          .catch(err => {
            console.log("err:", err);
          });
      },


      pushAnswer() {
        this.$store.commit("pushAnswer");


      },
      choiceKanji(num) {




        this.$store.commit('userChoice', this.kanji[num - 1]);


        this.$store.commit('checkAnswer', {
          userAnswer: this.kanji[num - 1]
        })

        if (this.i < 2) {
          this.kanji = "";
          this.moji = "";
          this.i++;
          this.$refs.reset.reset();
        }

      },
      // URLにパラメータとしてgradeが必要
      getQuestionAndAnswer() {
        const POST_URL = process.env.VUE_APP_URL_BASE + "question.py";
        // process.env.VUE_APP_URL_
        let params = new URLSearchParams();
        params.append("grade", this.$route.query.grade);
        //ここにURL指定。

        axios
          .post(POST_URL, params)
          .then(response => {
            console.log(response)
            this.$store.commit('initQuestionAndAnswer', response.data);
          })
          .catch(err => {
            console.log("err:", err);
          });

      },
    }
  };
</script>

<style scoped>
    h1 {
        position: relative;
    }

    h1:after {
        content: "";
        position: absolute;
        left: 0;
        bottom: 0;
        width: 100%;
        height: 7px;
        background: -webkit-repeating-linear-gradient(
                -45deg,
                #ffba73,
                #ffba73 2px,
                #fff 2px,
                #fff 4px
        );
        background: repeating-linear-gradient(
                -45deg,
                #ffba73,
                #ffba73 2px,
                #fff 2px,
                #fff 4px
        );
    }
</style>
