<script>
export default {
    name: "QuestionSelected1",
    props: {
        title: {
            type: String,
            default: "question text"
        },
        choices: {
            type: Array,
            default: () => {
                return ["是", "否"];
            }
        },
        id: {
            type: Number,
            default: 1
        }
    }
}
</script>

<template>
    <div class="main">
        <div class="title">
            {{ id }}. {{ title }}
        </div>
        <div class="box">
            <div style="grid-area: 1 / 2 / 2 / 3; opacity: 0;" class="wo">{{ choices[0] }}</div>
            <div style="grid-area: 1 / 3 / 2 / 4;">
                只赞同左边
            </div>
            <div style="grid-area: 1 / 4 / 2 / 5;">
                更赞同左边
            </div>
            <div style="grid-area: 1 / 5 / 2 / 6;">
                更赞同右边
            </div>
            <div style="grid-area: 1 / 6 / 2 / 7;">
                只赞同右边
            </div>
            <div style="grid-area: 1 / 7 / 2 / 8; opacity: 0;" class="wo">{{ choices[1] }}</div>
        </div>
        <div class="box">
            <div style="grid-area: 1 / 2 / 2 / 3;" class="wo">{{ choices[0] }}</div>
            <div style="grid-area: 1 / 3 / 2 / 4;" class="sel">
                <label for="q1a1">
                    <input type="radio" required value="-2" id="q1a1" :name="`bipolar_q${id}`">
                </label>
            </div>
            <div style="grid-area: 1 / 4 / 2 / 5;" class="sel">
                <label for="q1a2">
                    <input type="radio" required value="-1" id="q1a2" :name="`bipolar_q${id}`">
                </label>
            </div>
            <div style="grid-area: 1 / 5 / 2 / 6;" class="sel">
                <label for="q1a3">
                    <input type="radio" required value="1" id="q1a3" :name="`bipolar_q${id}`">
                </label>
            </div>
            <div style="grid-area: 1 / 6 / 2 / 7;" class="sel">
                <label for="q1a4">
                    <input type="radio" required value="2" id="q1a4" :name="`bipolar_q${id}`">
                </label>
            </div>
            <div style="grid-area: 1 / 7 / 2 / 8;" class="wo">{{ choices[1] }}</div>
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
        margin: 0 0 50px 0;
        text-align: left;
        text-indent: 1em;
        line-height: 1.6em;
    }
    .box {
        display: grid;
        grid-template-columns: 20px 1fr repeat(4, 2fr) 1fr 20px;
        justify-items: center;
        align-items: center;
        margin: 20px 0;
        font-size: 20px;
    }
    .box .wo {
        max-width: 100px;
        word-break: break-all;
    }
    .box .sel {
        width: 32px;
        height: 32px;
        border: 1px solid white;
        border-radius: 50%;
    }
    .box .sel label {
        display: block;
        width: 100%;
        height: 100%;
    }
    label>input {
        opacity: 0;
    }
    .box .sel:has(input[type=radio]:checked) {
        background-color: white;
    }
</style>