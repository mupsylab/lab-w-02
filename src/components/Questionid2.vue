<script>
import $ from 'jquery';
import 'jquery-ui-dist/jquery-ui';
import 'jquery-ui-dist/jquery-ui.min.css';

export default {
    name: "QuestionSelected2",
    props: {
        id: {
            type: Number,
            default: 1
        },
        title: {
            type: String,
            default: "question text"
        },
        choices: {
            type: Array,
            default: () => {
                return ["是", "否"];
            }
        }
    },
    data() {
        return {
            fRes: this.$props.choices,
            active: ["", "", "", ""]
        }
    },
    methods: {
        input_re(e) {
            e.target.value = e.target._value;
        }
    },
    setup() {
        return {}
    },
    created() {

    },
    mounted() {
        $(".wo").on("mouseup", (e) => {
            if ($(".wo[draging=yes]").length) {
                $("input").each((i, v) => {
                    v.setCustomValidity('')
                });
                const from = $(".wo[draging=yes]").data().id - 1;
                const to = $(e.target).data().id - 1;
                let newArr = [];
                this.fRes.forEach((v, i) => {
                    if ((i - from) * (i - to) > 0) {
                        newArr.push(v);
                    } else if (i == to) {
                        newArr.push(this.fRes[from]);
                    } else if (from < to) {
                        newArr.push(this.fRes[i + 1]);
                    } else if (from > to) {
                        newArr.push(this.fRes[i - 1]);
                    }
                });
                this.fRes = newArr;
                this.active = newArr;
            }
        });
        $(".wo").draggable({
            axis: "y",
            containment: ".box",
            start: (e) => {
                $(e.target).attr("draging", "yes");
            },
            drag: (e) => {
                $(e.target).css({
                    zIndex: 1
                });
            },
            stop: (e) => {
                $(e.target).css({
                    left: "0px",
                    top: "0px",
                    zIndex: 2
                });
                $(e.target).attr("draging", "no");
            }
        });
    }
}
</script>

<template>
    <div style="position: relative; top: -50px;">
        请拖动选项，按照重要性由高到低排序至满意。
    </div>
    <div class="main">
        <div class="title">
            {{ id }}. {{ title }}
        </div>
    </div>
    <div class="box">
        <div class="r" style="grid-area: 1 / 2 / 2 / 10;">
            <div class="num">1</div>
            <div class="wo" data-id="1">{{ fRes[0] }}</div>
            <input type="text" :name="`sort_q${id}_1`" id="a1" :value="active[0]" required @input="input_re" oninvalid="setCustomValidity('请拖动左边的选项进行排序')" />
        </div>
        <div class="r" style="grid-area: 2 / 2 / 3 / 10;">
            <div class="num">2</div>
            <div class="wo" data-id="2">{{ fRes[1] }}</div>
            <input type="text" :name="`sort_q${id}_2`" id="a2" :value="active[1]" required @input="input_re" oninvalid="setCustomValidity('请拖动左边的选项进行排序')" />
        </div>
        <div class="r" style="grid-area: 3 / 2 / 4 / 10;">
            <div class="num">3</div>
            <div class="wo" data-id="3">{{ fRes[2] }}</div>
            <input type="text" :name="`sort_q${id}_3`" id="a3" :value="active[2]" required @input="input_re" oninvalid="setCustomValidity('请拖动左边的选项进行排序')" />
        </div>
        <div class="r" style="grid-area: 4 / 2 / 5 / 10;">
            <div class="num">4</div>
            <div class="wo" data-id="4">{{ fRes[3] }}</div>
            <input type="text" :name="`sort_q${id}_4`" id="a4" :value="active[3]" required @input="input_re" oninvalid="setCustomValidity('请拖动左边的选项进行排序')" />
        </div>
    </div>
</template>

<style scoped>
.main {
    max-width: 75vw;
    font-size: 24px;
    line-height: 1.5em;
    user-select: none;
}

.title {
    font-size: 32px;
    text-align: left;
    text-indent: 2em;
    line-height: 1.6em;
}

.box {
    display: grid;
    grid-template-columns: 20px repeat(8, 1fr) 20px;
    grid-template-rows: repeat(4, 1fr);
    justify-items: stretch;
    text-align: left;
    font-size: 24px;
    line-height: 1.5em;
}

.box .r {
    position: relative;
}

.box .r .num {
    display: inline-block;
    padding: 5px 10px;
    margin: 5px 10px;
    font-size: 20px;
    line-height: 1.6em;
    border: 1px solid white;
    border-radius: 5px;
    vertical-align: middle;
}

.box .r .wo {
    display: inline-block;
    width: calc(100% - 60px);
    z-index: 2;
    cursor: pointer;
    box-sizing: border-box;
    border: 1px solid #e3e3e3;
    padding: 4px 10px;
    border-radius: 12px;
    vertical-align: middle;
}

.box .r input {
    position: absolute;
    top: 0;
    left: 0;
    opacity: 0;
}
</style>